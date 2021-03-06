# Using private beta services with boto3

You've been invited to the private beta of a new AWS service, 'darkspeed'. Boto3 doesn't have that service yet, but AWS has given you a json file with the service definition (`darkspeed-2018-07-06.normal.json`). How do you access it in boto3?

1. Rename the service definition file to `service-2.json` and put it in the following directory structure:

```plaintext
my_project /
  - __init__.py
  - my_program.py
  - darkspeed /
    - 2018-07-06 /
      - service-2.json
```

2. Set the environment variable `AWS_DATA_PATH` to point to `my_project` (do this before you call `boto3.client`). You can do this in `my_program.py`:

```python
os.environ['AWS_DATA_PATH'] = os.path.dirname(os.path.realpath(__file__))
```

3. Use the following boto3 call to create the client:

```python
darkspeed = boto3.client(
    service_name = 'darkspeed',
    region = 'us-east-9000'
)
```

Now you can call the service as usual:

```python
velocity = darkspeed.get_maximum_velocity(CharacterName = 'Sonic', Location = 'Green Hill Zone')
print(velocity)
```

## FAQ

### Does it have to be called `service-2.json`?

Yes. `service-1` is the old version, all closed betas I'm aware of are `service-2`.

### Does it have to have that exact directory structure?

Yes. `service_name/YYYY-MM-DD/service-2.json`. Check inside the json file. The service name has to match the `signingName` and the date has to match the `apiVersion` date. 

### What if I need to provide a specific endpoint URL?

You shouldn't need to, but if you do, use this:

```python
darkspeed = boto3.client(
    service_name = 'darkspeed',
    region_name = 'us-east-9000',
    endpoint_url = 'https://darkspeed-beta.us-east-9000.amazonaws.com'
)
```

### What if AWS gave me more than 1 file?

They had better given them to you named: `service-2.json`, `paginators-1.json`, `api-2.json`. If they didn't, check the documentation they gave you or ask your rep.

#### References

* [Loaders reference -- found this AFTER I wrote this whole thing](https://botocore.amazonaws.com/v1/documentation/api/latest/reference/loaders.html)
* [Boto3 session reference](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/core/session.html)
