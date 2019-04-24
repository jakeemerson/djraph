# Djraph OGM for the [TinkerPop 3](http://tinkerpop.apache.org/) [Gremlin Server](http://tinkerpop.apache.org/docs/current/reference/#gremlin-server)
[![Build Status](https://travis-ci.org/davebshow/djraph.svg?branch=master)](https://travis-ci.org/davebshow/djraph) [![Coverage Status](https://coveralls.io/repos/github/davebshow/djraph/badge.svg?branch=master)](https://coveralls.io/github/davebshow/djraph?branch=master) [![Gitter chat](https://badges.gitter.im/davebshow/djraph.svg)](https://gitter.im/davebshow/djraph?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge) [![Documentation Status](https://readthedocs.org/projects/djraph/badge/?version=latest)](http://djraph.readthedocs.io/en/latest/?badge=latest)


[Official Documentation](http://djraph.readthedocs.io/en/latest/)


Developers note:
The original Djraph was a TinkerPop 3 ready port of Cody Lee's mogwai, an excellent library that had been developed for use with pre-TinkerPop 3 versions of Titan. We designed Djraph to provide asynchronous programming abstractions that would work using any version of Python 2.7 + with a variety of asynchronous I/O libraries (Tornado, Asyncio, Trollius). While in theory this was great, we found that in our effort to promote compatibility we lost out on many of the features the newer Python versions provide to help developers deal with asynchronous programming. Our code base became large and made heavy use of callbacks, and nearly all methods and functions returned some sort of `Future`. This created both a clunky user API, and a code base that was difficult to reason about and maintain.

So, we decided to rewrite Djraph from scratch...

## Features

- High level asynchronous *Object Graph Mapper* (OGM)

- Integration with the *official gremlin-python Gremlin Language Variant* (GLV) - now provided by [aiogremlin](http://aiogremlin.readthedocs.io/en/latest/)

- Native Python support for asynchronous programing including *coroutines*,
  *iterators*, and *context managers* as specified in [PEP 492](https://www.python.org/dev/peps/pep-0492/)

- *Asynchronous Python driver* for the Gremlin Server - now provided by [aiogremlin](http://aiogremlin.readthedocs.io/en/latest/)

- Async `Graph` implementation that produces *native Python GLV traversals* - now provided by [aiogremlin](http://aiogremlin.readthedocs.io/en/latest/)
