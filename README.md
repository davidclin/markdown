# Markdown Sandbox
This is an area to experiment with GitHub markdown.
 
 
<details>
<summary>"Table of Contents"</summary>
Anchors<br>
Text<br>
Headers<br>
Carriage Returns<br>
Pre-formatted Text<br>
Spacing<br>
Bullets<br>
Tables<br>
Paragraphs<br>
Source Code<br>
Images<br>
Resources<br> 
</details>

<details>
<summary>"Table of Contents"</summary>
<pre>
(c7n_mailer) [ec2-user@ip-10-100-0-195 custodian]$ c7n-mailer --config mailer_sandbox.yml --update-lambda && custodian run -c security-groups-unused-slack.yml
Traceback (most recent call last):
  File "/home/ec2-user/c7n_mailer/bin/c7n-mailer", line 11, in <module>
    load_entry_point('c7n-mailer', 'console_scripts', 'c7n-mailer')()
  File "/home/ec2-user/cloud-custodian/tools/c7n_mailer/c7n_mailer/cli.py", line 131, in main
    mailer_config = get_and_validate_mailer_config(args)
  File "/home/ec2-user/cloud-custodian/tools/c7n_mailer/c7n_mailer/cli.py", line 95, in get_and_validate_mailer_config
    jsonschema.validate(config, CONFIG_SCHEMA)
  File "/home/ec2-user/c7n_mailer/local/lib/python2.7/site-packages/jsonschema/validators.py", line 541, in validate
    cls(schema, *args, **kwargs).validate(instance)
  File "/home/ec2-user/c7n_mailer/local/lib/python2.7/site-packages/jsonschema/validators.py", line 130, in validate
    raise error
jsonschema.exceptions.ValidationError: Additional properties are not allowed ('slack_token' was unexpected)

Failed validating u'additionalProperties' in schema:
    {u'additionalProperties': False,
     u'properties': {u'account_emails': {u'type': u'object'},
                     u'cache_engine': {u'type': u'string'},
                     u'contact_tags': {u'items': {u'type': u'string'},
                                       u'type': u'array'},
                     u'cross_accounts': {u'type': u'object'},
                     u'datadog_api_key': {u'type': u'string'},
                     u'datadog_application_key': {u'type': u'string'},
                     u'dead_letter_config': {u'type': u'object'},
                     u'debug': {u'type': u'boolean'},
                     u'from_address': {u'type': u'string'},
                     u'http_proxy': {u'type': u'string'},
                     u'https_proxy': {u'type': u'string'},
                     u'lambda_description': {u'type': u'string'},
                     u'lambda_name': {u'type': u'string'},
                     u'lambda_schedule': {u'type': u'string'},
                     u'lambda_tags': {u'type': u'object'},
                     u'ldap_bind_dn': {u'type': u'string'},
                     u'ldap_bind_password': {u'type': u'string'},
                     u'ldap_bind_password_in_kms': {u'type': u'boolean'},
                     u'ldap_bind_user': {u'type': u'string'},
                     u'ldap_email_attribute': {u'type': u'string'},
                     u'ldap_email_key': {u'type': u'string'},
                     u'ldap_manager_attribute': {u'type': u'string'},
                     u'ldap_uid_attribute': {u'type': u'string'},
                     u'ldap_uid_regex': {u'type': u'string'},
                     u'ldap_uid_tags': {u'items': {u'type': u'string'},
                                        u'type': u'array'},
                     u'ldap_uri': {u'type': u'string'},
                     u'memory': {u'type': u'integer'},
                     u'profile': {u'type': u'string'},
                     u'queue_url': {u'type': u'string'},
                     u'redis_host': {u'type': u'string'},
                     u'redis_port': {u'type': u'integer'},
                     u'region': {u'type': u'string'},
                     u'role': {u'type': u'string'},
                     u'runtime': {u'type': u'string'},
                     u'security_groups': {u'items': {u'type': u'string'},
                                          u'type': u'array'},
                     u'ses_region': {u'type': u'string'},
                     u'smtp_password': {u'type': u'string'},
                     u'smtp_port': {u'type': u'integer'},
                     u'smtp_server': {u'type': u'string'},
                     u'smtp_ssl': {u'type': u'boolean'},
                     u'smtp_username': {u'type': u'string'},
                     u'subnets': {u'items': {u'type': u'string'},
                                  u'type': u'array'},
                     u'timeout': {u'type': u'integer'}},
     u'required': [u'queue_url', u'role'],
     u'type': u'object'}

On instance:
    {'contact_tags': ['OwnerContact', 'OwnerEmail', 'SNSTopicARN'],
     'from_address': 'email@address.com',
     'queue_url': 'https://sqs.us-east-1.amazonaws.com/xxxxxxxxxxxx/david-lin-sandbox-cloudcustodian',
     'region': 'us-east-1',
     'role': 'arn:aws:iam::xxxxxxxxxxxx:role/CloudCustodianRole',
     'slack_token': 'xoxb-slack_token'}
</pre>
</details>

# Anchors
[Jump to anchor named 'text'](#text)<br>
[Jump to anchor named 'headers'](#headers)<br>
[Jump to anchor named 'carriage_returns'](#carriage_returns)<br>
and so on...

```
[Jump to anchor named 'foo'](#foo)
...
# <a name="foo"></a>Header goes here

```

# <a name="text"></a>Text
This is plain text.\
This is *italisized*.\
This is **bolded**\
This is ***bolded and italisized***

```
This is plain text.\
This is *italisized*.\
This is **bolded**\
This is ***bolded and italisized***
```


# <a name="headers"></a>     Header 1
##     Header 2
###    Header 3
####   Header 4
#####  Header 5
###### Header 6

```
#      Header 1
##     Header 2
###    Header 3
####   Header 4
#####  Header 5
###### Header 6
```

# <a name="carriage_returns"></a>Carriage Returns
This is a really long sentence that just got cut off\
OUCH!

```
This is a really long sentence that just got cut off\
OUCH!
```

# <a name="pre-formatted_text"></a>Pre-formatted Text
<pre>
This is pre-formatted text.
No breaks, paragraphs, or backslashes are used to produce a carriage return.
What you see is what you get.
</pre>

# <a name="spacing"></a>Spacing
The beginning of this sentence has no spaces followed by a backslash \(\\\).\
 The beginning of this sentence has 1 space followed by a backslash \(\\\).\
  The beginning of this sentence has 2 spaces followed by a backslash \(\\\).\
   The beginning of this sentence has 3 spaces followed by a backslash \(\\\).\
    The beginning of this sentence has 4 spaces with NO backslash at the end.

Spacing -- doesn't do jack. Everything gets aligned to the left.

```
The beginning of this sentence has no spaces followed by a backslash \(\\\).\
 The beginning of this sentence has 1 space followed by a backslash \(\\\).\
  The beginning of this sentence has 2 spaces followed by a backslash \(\\\).\
   The beginning of this sentence has 3 spaces followed by a backslash \(\\\).\
    The beginning of this sentence has 4 spaces with NO backslash at the end.
```

# <a name="bullets"></a>Bullets
* Bullet 1
* Bullet 2
  * Indent 1
    * Indent 2

```
* Bullet 1
* Bullet 2
  * Indent 1
    * Indent 2
```


# <a name="hyperlinks"></a>Hyperlinks 
[Example_1]: https://cloud.google.com \
[Example_2]: https://cloud.google.com/compute/docs/vpn/overview \
[Example_3](https://cloud.google.com/sdk/gcloud/reference/compute/routers/update-bgp-peer) \
Go to the [Example 4](https://console.cloud.google.com/networking/firewalls) page 

```
[Example_1]: https://cloud.google.com \
[Example_2]: https://cloud.google.com/compute/docs/vpn/overview \
[Example_3](https://cloud.google.com/sdk/gcloud/reference/compute/routers/update-bgp-peer) \
Go to the [Example 4](https://console.cloud.google.com/networking/firewalls) page 
```

# <a name="tables"></a>Tables

| key | value |
------|--------
| apple | green |
| cherry | red |


```
| key | value |
------|--------
| apple | green |
| cherry | red |
```

# <a name="paragraphs"></a>Paragraphs
1. First Paragraph \ 
   This is the first sentence.\ 
   This is the second sentence. \

1. Second Paragraph \
   This is the third sentence. \
   This is the fourth sentence. \

```
1. First Paragraph \ 
   This is the first sentence.\ 
   This is the second sentence. \

1. Second Paragraph \
   This is the third sentence. \
   This is the fourth sentence. \
```

# <a name="source_code"></a>Source Code

```
$ yum install update
```

```python
print 'hello, world!'
# Comment
```


# <a name="images"></a>Images
![](./images/button.png)

<p><img align="left" src="./images/button.png" height="200" />Image aligned to the left</p>
<br>xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
<br>xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

<br>

<hr size = "10"> 

<br>

<p><img align="right" src="./images/button.png" height="200" />Image aligned to the right</p>
<br>xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
<br>xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

<br>

<hr size = "10">

<br><br>xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
<br>xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

<p><img align="center" src="./images/button.png" height="200" />Image aligned to the center</p>
<br>xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
<br>xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx


```
![](./images/button.png)
<p><img align="left" src="./images/button.png" height="200" />Image aligned to the left</p>
<hr size = "10"> 
<p><img align="right" src="./images/button.png" height="200" />Image aligned to the right</p>
<hr size = "10">
<p><img align="center" src="./images/button.png" height="200" />Image aligned to the center</p>
```

# <a name="resources"></a>Resources
