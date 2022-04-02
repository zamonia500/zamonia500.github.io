---
title: "What Is Context Manager"
date: 2022-04-02T15:54:19+09:00
draft: false
categories: ["python"]
tags: ["context manager"]
---

# Summary

- Context manager란? : runtime context를 위한 런타임 환경을 관리해주는 객체
- contextmanager decorator : context manager를 구현하기 쉽게 해주는 helper decorator

# With Statement Context Managers

- context manager는 `with` 문이 실행되었을 때, 생성되는 runtime context를 정의하는 객체이다.
- code block을 실행하는 데 필요한 entry / exit 로직을 담당한다.
- 보통 `with` 문으로 실행되지만, method를 직접 호출하여 사용할 수 있다.

```python
object.__enter__(self)
# Enter the runtime context related to this object. 
# The with statement will bind this method’s return value 
# to the target(s) specified in the as clause of the statement, if any.

object.__exit__(self, exc_type, exc_value, traceback)
# Exit the runtime context related to this object.
# The parameters describe the exception
# that caused the context to be exited. 
# If the context was exited without an exception,
# all three arguments will be None.

# If an exception is supplied,
# and the method wishes to suppress the exception
# (i.e., prevent it from being propagated),
# it should return a true value.
# Otherwise, the exception will be processed normally upon exit from this method.

# Note that __exit__() methods should not reraise the passed-in exception;
# this is the caller’s responsibility.
```

# The `with` statement

- [https://docs.python.org/3/reference/compound_stmts.html#the-with-statement](https://docs.python.org/3/reference/compound_stmts.html#the-with-statement)

`with` statement를 사용하는 1번 코드블록은 2번 코드블록과 같은 내용을 함축한다.

```python
with EXPRESSION as TARGET:
    SUITE
```

```python
manager = (EXPRESSION)
enter = type(manager).__enter__
exit = type(manager).__exit__
value = enter(manager)
hit_except = False

try:
    TARGET = value
    SUITE
except:
    hit_except = True
    if not exit(manager, *sys.exc_info()):
        raise
finally:
    if not hit_except:
        exit(manager, None, None, None)
```

# contextlib

- [https://docs.python.org/3/library/contextlib.html](https://docs.python.org/3/library/contextlib.html)

## @contextlib.contextmanager

- [https://docs.python.org/3/library/contextlib.html#contextlib.contextmanager](https://docs.python.org/3/library/contextlib.html#contextlib.contextmanager)
- context manager factory function을 정의할 때 사용하는 decorator

### contextmanager를 사용하는 이유 (주관적)

1. `__enter__`, `__exit__` 2개의 magic method를 정의하는 것 보다 코드가 간결하다.
    - without contextmanager
        
        ```python
        # https://blog.ramosly.com/python-context-managers-and-the-with-statement-8f53d4d9f87
        from sqlalchemy import create_engine
        from sqlalchemy.orm import sessionmaker
        
        class SQLAlchemyDBConnection(object):
            """SQLAlchemy database connection"""
            def __init__(self, connection_string):
                self.connection_string = connection_string
                self.session = None
        				self.engine = create_engine(self.connection_string)
                self.Session = sessionmaker()
        
            def __enter__(self):
                self.session = self.Session(bind=self.engine)
                return self
        
            def __exit__(self, exc_type, exc_val, exc_tb):
                self.session.close()
        ```
        
    - with context manager
        
        ```python
        from contextlib import contextmanager
        
        from sqlalchemy import create_engine
        from sqlalchemy.orm import sessionmaker
        
        class SQLAlchemyDBConnection(object):
            """SQLAlchemy database connection"""
            def __init__(self, connection_string):
                self.connection_string = connection_string
        				self.engine = create_engine(self.connection_string)
                self.Session = sessionmaker()
        
            @contextmanager
        		def session_factory(self):
        				try:
                    session = Session(bind=engine)
        						yield session
                finally:
                    session.close()
        ```
        
2. Default Context manager를 custom하여 사용할 수 있다.
    - Default
        
        ```python
        # https://blog.ramosly.com/python-context-managers-and-the-with-statement-8f53d4d9f87
        from sqlalchemy import create_engine
        from sqlalchemy.orm import sessionmaker
        
        class SQLAlchemyDBConnection(object):
            """SQLAlchemy database connection"""
            def __init__(self, connection_string):
                self.connection_string = connection_string
                self.session = None
        				self.engine = create_engine(self.connection_string)
                self.Session = sessionmaker()
        
            def __enter__(self):
                self.session = self.Session(bind=self.engine)
                return self
        
            def __exit__(self, exc_type, exc_val, exc_tb):
                self.session.close()
        ```
        
    - Custom

        ```python
        from contextlib import contextmanager

        from sqlalchemy import create_engine
        from sqlalchemy.orm import sessionmaker

        class SQLAlchemyDBConnection(object):
            """SQLAlchemy database connection"""
            def __init__(self, connection_string):
                self.connection_string = connection_string
                self.session = None
                        self.engine = create_engine(self.connection_string)
                self.Session = sessionmaker()

                @contextmanager
                def session_factory(self):
                        try:
                    session = Session(bind=engine)
                                yield session
                except NoResultFound as e:
                    session.rollback()
                    # Do some custom logic
                except MultipleResultsFound as e:
                    session.rollback()
                    # Do some custom logic
                except IntegrityError as e:
                    session.rollback()
                    # Do some custom logic
                finally:
                    session.close()
        ```
