Handling HTTP 429 errors in Python
==================================

This repo contains an example of how to handle HTTP 429 errors with Python.

The `HTTP 429 status code <https://tools.ietf.org/html/rfc6585#section-4>`_
means "Too Many Requests", and it's sent when a server wants you to slow
down the rate of requests.

The `tenacity library <https://github.com/jd/tenacity>`_ has some functions
for handling retry logic in Python.  This repo has an example of how to use
tenacity to retry requests that got an HTTP 429 status code.

The example uses `urllib.request
<https://docs.python.org/3/library/urllib.request.html>`_ from the standard
library, but this approach can be adapted for other HTTP libraries.

There are two functions in ``handle_http_429_errors.py``:

*  ``retry_if_http_429_error()`` can be used as the ``retry`` keyword of
   ``@tenacity.retry``.

   It retries a function if the function makes an HTTP request that returns
   a 429 status code.

*  ``wait_for_retry_after_header(underlying)`` can be used as the ``wait`` keyword
   of ``@tenacity.retry``.

   It looks for the Retry-After header in the HTTP response, and waits for that
   long if present.  If not, it uses the supplied fallback strategy.

In this example, our ``get_url()`` function tries to request a URL.  If it
gets an HTTP 429 response, it waits up to three times before erroring out --
either respecting the ``Retry-After`` header from the server, or 1 second between
consecutive requests if not.

.. code-block:: python

   import urllib.request

   from tenacity import retry, stop_after_attempt, wait_fixed

   from handle_http_429_errors import (
       retry_if_http_429_error,
       wait_for_retry_after_header
   )


   @retry(
       retry=retry_if_http_429_error(),
       wait=wait_for_retry_after_header(fallback=wait_fixed(1)),
       stop=stop_after_attempt(3)
   )
   def get_url(url):
       return urllib.request.urlopen(url)


   if __name__ == "__main__":
       get_url(url="https://httpbin.org/status/429")

Reader caution: I wrote this as a proof-of-concept in a single evening.  It's not
rigorously tested, but hopefully gives an idea of how you might do this if you
wanted to implement it properly.