# API_Documentation

##Introduction

Shift client API. All API endpoints are relative to the /api prefix.

All endpoints will return:

- Success parameter. true or false dependent on success.
- Error parameter. Provided when response is unsuccessful.

The API is only available after the client has successfully loaded, otherwise all endpoints will return:

```
{
    "success" : false,
    "error" : "loading"
}
```

In the case the client is not fully synced all routes may return intermediate/old values.

Each API entry contains an example call to help provide understanding of how to use the call.

These examples rely on ```curl``` being installed and Shift running on the localhost. 

The examples also include ```<field>;``` use this for easy identification of what needs to be changed for the call to function.
