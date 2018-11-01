# roku-fetch

Roku-fetch is a [Fetch]-inspired implementation for Roku/BrightScript. It provides a familiar API that abstracts away
the Roku-specific roUrlTransfer and roUrlEvent objects.

## Basic Usage

    response = fetch({url: "http://example.url"})
    if response.ok
        ?"The response was: " + response.text()
    end if

## Full Usage

    response = fetch({
        url: "http://example.url",
        timeout: 5000,
        method: "PUT",
        headers: {
            "Content-Type": "application/json",
            "If-None-Match": "abc123"
        }
        body: FormatJson({id: "xyz", amount: 8.29})
    })
    if response.ok
        etag = response.headers["ETag"].value
        cookies = response.headers["Set-Cookie"]
        while cookies <> invalid
            ?cookies.value
            cookies = cookies.next
        end while
        json = response.json()
        ?json.items.total
    else
        ?"The request failed", response.statusCode, response.text()
    end if

## Request Options

    options: {
        url:     [req] string - http or https url
        timeout: [opt] int - ms to wait before timeout (defaults to 0 (no timeout))
        headers: [opt] assocarray - list of request headers where key=headername and val=headervalue
        method:  [opt] string - the HTTP method. GET|POST|PUT|DELETE|etc (defaults to GET)
                       if you are doing a normal GET or POST, you can omit this - it is only useful for the other verbs
        body:    [opt] string - the preformatted request body (ie: form data). 
                       if specified, the request method will default to POST unless overridden with options.method
    }

## Response Object

    returns Response object: {
        status:  int - the HTTP status code (ex: 200); See 'Status Codes' section below
        ok:      bool - true if the response is successful (200-299)
        headers: assocarray - where each header name is a key; See 'Response Headers' section below
        text():  string - function that returns the raw string response
        json():  object - function that returns the response parsed as JSON
        xml():   object - function that returns the response parsed as an roXmlElement
    }

#### Status Codes

The actual HTTP status code is returned. However, in cases where the Roku framework could not even send the request at all,
the underlying cUrl error code will be returned instead. You can differentiate between HTTP and cUrl status codes because all
cUrl error codes are negative. See the [roUrlEvent] documenation for a full list of error codes.

For cases where the request cannot be sent and the cUrl error code is returned, the `body` will be set to the full error message returned from `GetFailureReason()` and can be accessed via `response.text()`.

#### Response Headers

Since HTTP allows for multiple headers with the same name, the response headers object is structured like this:

    {
        "header_name": headerValueObject
    }

where headerValueObject looks like:

    {
        "value": "value of the header",
        "next": pointer to the next headerValueObject for this same header (or invalid)
    }

Single value example:

    contentType = response.headers["Content-Type"].value

Multiple value example:

    cookies = response.headers["Set-Cookie"]
    while cookies <> invalid
        ?cookies.value
        cookies = cookies.next
    end while

## Other Notes

* Since this library relies on the roUrlTransfer object behind the scenes, it can only be used from a Task node.

* It is designed for the most common use case where tasks are created, do some work, return results, and then 
go out of scope. If your tasks are long-lived and service multiple requests, they likely already have their own 
message port and a different pattern is recommended (though you can 'split' the Fetch implementation into the 
request-sending portion and the response-handling portion and hook into your existing message port)

[Fetch]: https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch]
[roUrlEvent]: https://sdkdocs.roku.com/display/sdkdoc/roUrlEvent
