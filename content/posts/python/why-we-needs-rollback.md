---
title: "Why We Needs Rollback"
date: 2022-03-31T23:16:19+09:00
draft: true
categories: ["python"]
tags: ["SQLAlchemy"]
---

# 어디에서나 보이는 Example code

다음과 같은 try-except-finally 코드를 굉장히 흔하게 볼 수 있다.
session 내부에서 실행한 코드에 문제가 없다면 `commit`을 호출하고,
session 내부에서 except가 발생했을 경우 `rollback`을 호출한다.
그리고 마지막에는 `close`를 호출한다.

[#when-do-i-construct-a-session-when-do-i-commit-it-and-when-do-i-close-it](https://docs.sqlalchemy.org/en/13/orm/session_basics.html#when-do-i-construct-a-session-when-do-i-commit-it-and-when-do-i-close-it)
```python
from contextlib import contextmanager

@contextmanager
def session_scope():
    """Provide a transactional scope around a series of operations."""
    session = Session()
    try:
        yield session
        session.commit()
    except:
        session.rollback()
        raise
    finally:
        session.close()


def run_my_program():
    with session_scope() as session:
        ThingOne().go(session)
        ThingTwo().go(session)
```

[Python dependency injector example](https://python-dependency-injector.ets-labs.org/examples/fastapi-sqlalchemy.html#database)
```python
"""Database module."""

from contextlib import contextmanager, AbstractContextManager
from typing import Callable
import logging

from sqlalchemy import create_engine, orm
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import Session

logger = logging.getLogger(__name__)

Base = declarative_base()


class Database:

    def __init__(self, db_url: str) -> None:
        self._engine = create_engine(db_url, echo=True)
        self._session_factory = orm.scoped_session(
            orm.sessionmaker(
                autocommit=False,
                autoflush=False,
                bind=self._engine,
            ),
        )

    def create_database(self) -> None:
        Base.metadata.create_all(self._engine)

    @contextmanager
    def session(self) -> Callable[..., AbstractContextManager[Session]]:
        session: Session = self._session_factory()
        try:
            yield session
        except Exception:
            logger.exception("Session rollback because of exception")
            session.rollback()
            raise
        finally:
            session.close()
```


# 각각의 역할들을 톹아보자

try-except-finally에서 사용된 session method는 `commit`, `rollback`, `close` 3개이다.
이 method들이 어떤 역할을 하는지 톧아보자.

## commit
[Committing](https://docs.sqlalchemy.org/en/13/orm/session_basics.html#committing)

 `Session.commit()`은 현재 transation을 commit 하기 위해 사용된다. "autoflush" 세팅과는 상관없이, 항상`Session.flush()`를 호출한다. transaction이 없다면, 에러를 raise 한다.
  Session은 기본적으로 항상 "transaction"을 사용하는데, `autocommit=True` 으로 설정하여 transaction의 사용을 disable 할 수 있다. autocommit mode에서는 `Session.begin()` 을 사용하여 명시적으로 transaction을 시작해야 한다.

## Rollback
[Rolling Back](https://docs.sqlalchemy.org/en/13/orm/session_basics.html#rolling-back)

 `Session.rollback()` 은 현재 transaction을 rollback한다. rollback 후의 session 상태는 다음과 같다.
 - 모든 transaction이 rollback 됨, session이 connection에 직접 연결되어 있지 않다면 모든 connection들이 반환 됨(connection에 직접 연결되어 있다고 해도 connection을 살아있지만 rollback은 수행됨).
 - INSERT 문의 transaction에서 세션에 추가된 `Pending` 상태의 객체는 INSERT 문이 rollback 될 때 삭제된다.
 - DELETE 문의 transaction에서 rollback이 일어날 경우 `Deleted` -> `Persistent` 상태로 바뀜.

> When a [`Session.flush()`](https://docs.sqlalchemy.org/en/13/orm/session_api.html#sqlalchemy.orm.session.Session.flush "sqlalchemy.orm.session.Session.flush") fails, typically for reasons like primary key, foreign key, or “not nullable” constraint violations, a [`Session.rollback()`](https://docs.sqlalchemy.org/en/13/orm/session_api.html#sqlalchemy.orm.session.Session.rollback "sqlalchemy.orm.session.Session.rollback") is issued automatically (it’s currently not possible for a flush to continue after a partial failure). However, the flush process always uses its own transactional demarcator called a _subtransaction_, which is described more fully in the docstrings for [`Session`](https://docs.sqlalchemy.org/en/13/orm/session_api.html#sqlalchemy.orm.session.Session "sqlalchemy.orm.session.Session"). What it means here is that even though the database transaction has been rolled back, the end user must still issue [`Session.rollback()`](https://docs.sqlalchemy.org/en/13/orm/session_api.html#sqlalchemy.orm.session.Session.rollback "sqlalchemy.orm.session.Session.rollback") to fully reset the state of the [`Session`](https://docs.sqlalchemy.org/en/13/orm/session_api.html#sqlalchemy.orm.session.Session "sqlalchemy.orm.session.Session").

## Closing
[Closing](https://docs.sqlalchemy.org/en/13/orm/session_basics.html#closing)

- Session.close()는 내부적으로 세션에 연결되어 있는 Session.expunge_all() method를 호출한다. 그리고 Engine object에 연결되어 있는 모든 transaction & connection을 release 한다.
- 다시 사용할 수 있다.


# 왜 rollback이 필요한 지 모르겠는 이유

- 



# 그럼에도 불구하고 Example code가 Best practice 인 이유
