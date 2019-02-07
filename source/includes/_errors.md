# Errors

The smaXtec Intern API uses the following error codes:


Error Code | Message | Description
---------- | ------- | -----------
401 | UnauthorizedError | The server could not verify that you are authorized to access the URL requested. You either supplied the wrong credentials (e.g. a bad password), or your browser doesn’t understand how to supply the credentials required.
404 | NotFoundError | Organisation not found.
409 | ConflictError | A conflict happened while processing the request. The resource might have been modified while the request was being processed.
422 | AttributeError | The request was well-formed but was unable to be followed due to semantic errors.
