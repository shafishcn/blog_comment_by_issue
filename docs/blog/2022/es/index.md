## ES���ĸ���
## ES��������
``` shell
# �����ĵ�
PUT /test/_doc/1
{
    "name":"shafish",
    "age":"1000"
}
```

``` json
{
  "_index" : "test",
  "_type" : "type1",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

``` shell
# ������������
PUT /test2
{
  "mappings":{
    "properties":{
      "name":{
        "type":"text"
      },
      "age":{
        "type":"long"
      }
    }
  }
}
```

``` json
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test2"
}
```
