---
title: "python aiohttp 사용법"
date: 2022-03-13T14:16:29+09:00
draft: false
tag: ["python", "aiohttp"]
category: ["python"]
---

# [aiohttp](https://docs.aiohttp.org/en/stable/)란?

aiohttp는 "Async HTTP client/server for asyncio and Python"이다.  
블로킹 콜(blocking-call)이 일어나는 [request](https://docs.python-requests.org/en/latest/)와는 다르게,
aiohttp를 사용하면 싱글 스레드 환경에서 코루틴을 활용한 병렬 처리를 할 수 있다.

aiohttp는 client / server를 위한 기능이 모두 있지만, 나는 client 기능만을 사용하였다.

# aiohttp 사용해서 KFP(Kubeflow Pipeline) API 호출하기

## KFP SDK가 있긴 하다

kfp(Kubeflow Pipeline) SDK는 자동 생성되는 녀석이라 사용성이 너무 좋지 않았다.
사용하고자 하는 kfp의 `upload_pipeline`, `upload_pipeline_version` 함수를 사용하고자 했는데, 이 함수들의 시그니쳐는 다음과 같다.

```python
# https://kubeflow-pipelines.readthedocs.io/en/stable/_modules/kfp/_client.html#Client.upload_pipeline
def upload_pipeline(
    self,
    pipeline_package_path: str = None,
    pipeline_name: str = None,
    description: str = None,
) -> kfp_server_api.ApiPipeline:
    """Uploads the pipeline to the Kubeflow Pipelines cluster.

     Args:
      pipeline_package_path: Local path to the pipeline package.
      pipeline_name: Optional. Name of the pipeline to be shown in the UI.
      description: Optional. Description of the pipeline to be shown in the UI.

     Returns:
      Server response object containing pipleine id and other information.
    """

    response = self._upload_api.upload_pipeline(
        pipeline_package_path, name=pipeline_name, description=description)
    if self._is_ipython():
        import IPython
        html = '<a href=%s/#/pipelines/details/%s>Pipeline details</a>.' % (
            self._get_url_prefix(), response.id)
        IPython.display.display(IPython.display.HTML(html))
    return response
```

```python
def upload_pipeline_version(
    self,
    pipeline_package_path,
    pipeline_version_name: str,
    pipeline_id: Optional[str] = None,
    pipeline_name: Optional[str] = None,
    description: Optional[str] = None,
) -> kfp_server_api.ApiPipelineVersion:
    """Uploads a new version of the pipeline to the Kubeflow Pipelines cluster.

    Args:
      pipeline_package_path: Local path to the pipeline package.
      pipeline_version_name:  Name of the pipeline version to be shown in the UI.
      pipeline_id: Optional. Id of the pipeline.
      pipeline_name: Optional. Name of the pipeline.
      description: Optional. Description of the pipeline version to be shown in the UI.

    Returns:
      Server response object containing pipleine id and other information.

    Raises:
      ValueError when none or both of pipeline_id or pipeline_name are specified
      kfp_server_api.ApiException: If pipeline id is not found.
    """

    if all([pipeline_id, pipeline_name
           ]) or not any([pipeline_id, pipeline_name]):
        raise ValueError('Either pipeline_id or pipeline_name is required')

    if pipeline_name:
        pipeline_id = self.get_pipeline_id(pipeline_name)
    kwargs = dict(
        name=pipeline_version_name,
        pipelineid=pipeline_id,
    )

    if description:
        kwargs['description'] = description
    try:
        response = self._upload_api.upload_pipeline_version(
            pipeline_package_path, **kwargs)
    except kfp_server_api.exceptions.ApiTypeError as e:
        # ToDo: Remove this once we drop support for kfp_server_api < 1.7.1
        if 'description' in e.message and 'unexpected keyword argument' in e.message:
            raise NotImplementedError(
                'Pipeline version description is not supported in current kfp-server-api pypi package. Upgrade to 1.7.1 or above'
            )
        else:
            raise e

    if self._is_ipython():
        import IPython
        html = '<a href=%s/#/pipelines/details/%s>Pipeline details</a>.' % (
            self._get_url_prefix(), response.id)
        IPython.display.display(IPython.display.HTML(html))
    return response
```

두 함수 모두 `pipeline_package_path`를 전달받는다. 즉 파일 객체 (file-like object)가 아니라 실제 파일의 경로를
입력받는다는 말이였다. 실제 파일의 경로를 사용하기 힘들고, 리소스를 더 효율적으로 사용하기 위해 aiohttp를 통해 API를 호출하기로 했다.


## KFP API Spec

직접 http request를 만들어 보내기로 한 뒤 API spec을 확인해 보았다. [link](https://www.kubeflow.org/docs/components/pipelines/reference/api/kubeflow-pipeline-api-spec/#tag-PipelineService)에서 확인 할 수 있다.

![](/images/posts/upload.png)
![](/images/posts/upload-version.png)

# Build aiohttp request

API spec에 따라 다음과 같이 request를 구성하면 된다.

1. Content-Type: multipart/form-data
2. formData의 uploadfile key에 파일 정보를 넣어준다
3. form-data의 구분을 위해 boundary를 설정해 주어야 한다
4. name, description은 query parameter이다

*파일 입출력도 aiofiles 같은 패키지를 사용하는 것이 좋지만 이 포스트의 범위를 벗어나므로 파일 읽기에는 코루틴을 사용하지 않았다*

## `POST /apis/v1beta1/pipelines/upload`
```python
import asyncio
from typing import Final
from pathlib import Path
from uuid import uuid4

from aiohttp import ClientSession, MultipartWriter


# http request를 대신 받아줘서 확인할 수 있게 해 주는 사이트
WEBHOOK_SITE: Final = "https://webhook.site/0c52884c-e9d1-4814-aa78-000000000000"  
PIPELINE_UPLOAD_ENDPOINT: Final = "/apis/v1beta1/pipelines/upload"
ENDPOINT: Final = f"{WEBHOOK_SITE}/{PIPELINE_UPLOAD_ENDPOINT}"
BOUNDARY: Final = f"--AIO-HTTP-{uuid4()}"


async def upload_pipeline(name: str, pipeline_file: str):

    with open(pipeline_file) as f:
        uploadfile = f.read()

    with MultipartWriter(boundary=BOUNDARY) as mpwriter:
        part = mpwriter.append(uploadfile)
        part.set_content_disposition(
            "form-data",
            name="uploadfile",
            filename="PIPELINE.yaml",
        )

    async with ClientSession() as session:
        query_params = {"name": name, "description": ""}
        headers = {
            "Accept": "application/json",
            "Content-Type": f"multipart/form-data;boundary={BOUNDARY}",
        }
        async with session.post(ENDPOINT,
                                params=query_params,
                                data=mpwriter,
                                headers=headers,
        ) as r:
            response_json = await r.json()

asyncio.run(upload_pipeline, "New Pipeline", "New Pipeline.yaml")
```

![](/images/posts/upload-webhook-result.png)

# 테스트 하는 법
