# `pcluster list-image-log-streams`<a name="pcluster.list-image-log-streams-v3"></a>

Retrieve the list of log streams associated with an image\.

```
pcluster list-image-log-streams [-h] --image-id IMAGE_ID
                                [--region REGION]
                                [--next-token NEXT_TOKEN] [--debug]
                                [--query QUERY]
```

## Named arguments<a name="pcluster-v3.list-image-log-streams.namedargs"></a>

`-h, --help`  
Shows the help text for `pcluster list-image-log-streams`\.

`--image-id IMAGE_ID`  
Specifies the ID of the image\.

`--region REGION`  
Specifies the AWS Region to use\. The Region must be specified, either using the `AWS_DEFAULT_REGION` environment variable or the `--region` parameter\.

`--next-token NEXT_TOKEN`  
Specifies the token to use for paginated requests\.

`--debug`  
Enables debug logging\.

`--query QUERY`  
Specifies the JMESPath query to perform on the output\.