# CRUD API

## APIs

### Read

```shell-session
eldeim@htb[/htb]$ curl http://<SERVER_IP>:<PORT>/api.php/city/london

[{"city_name":"London","country_name":"(UK)"}]
```

We can pipe the output to the `jq` utility, which will format it properly. We will also silent any unneeded cURL output with `-s`, as follows:

```shell-session
eldeim@htb[/htb]$ curl -s http://<SERVER_IP>:<PORT>/api.php/city/london | jq

[
  {
    "city_name": "London",
    "country_name": "(UK)"
  }
]
```

### Create

We can simply POST our JSON data, and it will be added to the table. As this API is using JSON data, we will also set the `Content-Type` header to JSON, as follows:

```shell-session
eldeim@htb[/htb]$ curl -X POST http://<SERVER_IP>:<PORT>/api.php/city/ -d '{"city_name":"HTB_City", "country_name":"HTB"}' -H 'Content-Type: application/json'
```

Now, we can read the content of the city we added (`HTB_City`), to see if it was successfully added:

```shell-session
eldeim@htb[/htb]$ curl -s http://<SERVER_IP>:<PORT>/api.php/city/HTB_City | jq

[
  {
    "city_name": "HTB_City",
    "country_name": "HTB"
  }
]
```

### Update

Using `PUT` is quite similar to `POST` in this case, with the only difference being that we have to specify the name of the entity we want to edit in the URL, otherwise the API will not know which entity to edit. So, all we have to do is specify the `city` name in the URL, change the request method to `PUT`, and provide the JSON data like we did with POST, as follows:

```shell-session
eldeim@htb[/htb]$ curl -X PUT http://<SERVER_IP>:<PORT>/api.php/city/london -d '{"city_name":"New_HB_City", "country_name":"HTB"}' -H 'Content-Type: application/json'
```

> Note: In some APIs, the `Update` operation may be used to create new entries as well. Basically, we would send our data, and if it does not exist, it would create it. For example, in the above example, even if an entry with a `london` city did not exist, it would create a new entry with the details we passed. In our example, however, this is not the case. Try to update a non-existing city and see what you would get.

### Delete

We simply specify the city name for the API and use the HTTP `DELETE` method, and it would delete the entry, as follows:

```shell-session
eldeim@htb[/htb]$ curl -X DELETE http://<SERVER_IP>:<PORT>/api.php/city/New_HTB_City
```

```shell-session
eldeim@htb[/htb]$ curl -s http://<SERVER_IP>:<PORT>/api.php/city/New_HTB_City | jq
[]
```

As we can see, after we deleted `New_HTB_City`, we get an empty array when we try reading it, meaning it no longer exists.
