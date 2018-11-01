# status
[![Build Status](https://travis-ci.org/arxanchain/tomago-sdk-java.svg?branch=master)](https://travis-ci.org/arxanchain/tomago-sdk-java)

# tomago-sdk-java

Tomago is a project code name, which is used to wrap SmartContract invocation
from the business point of view, including APIs for managing digital assets,
asset owners (entities), etc. management, digital assets, etc. You need not
care about how the backend blockchain runs or the unintelligible techniques,
such as consensus, endorsement and decentralization. Simply use the SDK we
provide to implement your business logics, we will handle the caching, tagging,
compressing, encrypting and high availability.

We also provide a way from this SDK to invoke the SmartContract, a.k.a.
Chaincode, which is deployed by yourself.

This SDK enables *Java* developers to develop applications that interact with the
SmartContract which is deployed out of the box or by yourself in the ArxanChain
BaaS Platform via Tomago.

# Usage
Tomago-sdk-java is Maven projcet, we have already put this project to Maven Repository.
When you use tomago-sdk-java, you should reference project like this:

```pom.xml```

```
<dependency>
    <groupId>com.arxanfintech</groupId>
    <artifactId>tomago-sdk-java</artifactId>
    <version>2.0.1</version>
</dependency>
```

## Sample Usage
Tomago-sdk-java have three import API. For more details please refer to [tomago api](http://www.arxanfintech.com/infocenter/html/development/mycc.html)

Before you use tomago-sdk-java, you should prepare certificates.

The certificates include:

* The public key of ArxanChain BaaS Platform (server.crt) which is used to
  encrypt the data sent to Tomago service. You can download it from the
  ArxanChain BaaS ChainConsole -> System Management -> API Certs Management
* The private key of the client user (such as `APIKey.key`) which is used to sign the
  data. You can download it when you create an API Certificate.

After downloading the two files, use the following command to convert your private key file into PEM format.

```sh
$ openssl ec -in apikey.key -outform PEM -out apikey.key
```

Then copy (rename as follows) your TLS certificate and PEM private key file as follows path:

```
└── your_cert_dir
    ├── tls
    |   └── tls.cert
    └── users
        └── your-api-key
            └── your-api-key.key
```

#### invoke
```java
import com.arxanfintech.sdk.tomago.Tomago;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;

String strdata = "{\"payload\": {\"chaincode_id\": \"pubchain-mycc\",\"args\": [\"invoke\", \"a\", \"b\", \"1\"]} }";
JSONObject jsondata = JSON.parseObject(strdata);

String strheader = "{\"Callback-Url\":\"http://something.com\", \"Channel-Id\":\"pubchain\"}";
JSONObject jsonheader = JSON.parseObject(strheader);

String address = "127.0.0.1:9143"; //YOUR SERVER ADDRESS
String api_key = "MitFg-7HM1540973852"; // API-KEY
String cert_path = "/home/arxan/bass/cert/"; //your_cert_dir
Client client = new Client(api_key, cert_path, "", "", "", "", address, true, "tomago");

Tomago tomago = new Tomago(client);
       
String response = tomago.Invoke(jsonheader, jsondata);    
```

#### query
```java
import com.arxanfintech.sdk.tomago.Tomago;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;

String strdata = "{\"payload\": {\"chaincode_id\": \"mycc\",\"args\":[\"query\", \"a\"]} }";
JSONObject jsondata = JSON.parseObject(strdata);

String strheader = "{\"Callback-Url\":\"http://something.com\", \"Channel-Id\":\"pubchain\"}";
JSONObject jsonheader = JSON.parseObject(strheader);

String address = "127.0.0.1:9143"; //YOUR SERVER ADDRESS
String api_key = "MitFg-7HM1540973852"; // API-KEY
String cert_path = "/home/arxan/bass/cert/"; //your_cert_dir
Client client = new Client(api_key, cert_path, "", "", "", "", address, true, "tomago");

Tomago tomago = new Tomago(client);
     
String response = tomago.Query(jsonheader, jsondata); 
```

#### transaction/{TXNID}
```java
import com.arxanfintech.sdk.tomago.Tomago;
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;

String id = "df24213a7ffa19c53f506c3dea8544df750905f5325fac8217496b24c5a880e8"; // TransactionId

String strheader = "{\"Callback-Url\":\"http://something.com\"}";
JSONObject jsonheader = JSON.parseObject(strheader);

String address = "127.0.0.1:9143"; //YOUR SERVER ADDRESS
String api_key = "MitFg-7HM1540973852"; // API-KEY
String cert_path = "/home/arxan/bass/cert/"; //your_cert_dir
Client client = new Client(api_key, cert_path, "", "", "", "", address, true, "tomago");

Tomago tomago = new Tomago(client);
        
String response = tomago.Transaction(jsonheader, id);

```

## Using callback URL to receive blockchain transaction events

Tomago is asynchronous, it will return without waiting for
blockchain transaction confirmation. In asynchronous mode, you should set
`Callback-Url` in the http header to receive blockchain transaction events.

The blockchain transaction event structure is defined as follows:

```code
import google_protobuf "github.com/golang/protobuf/ptypes/timestamp

// Blockchain transaction event payload
type BcTxEventPayload struct {
    BlockNumber   uint64                     `json:"block_number"`   // Block number
    BlockHash     []byte                     `json:"block_hash"`     // Block hash
    ChannelId     string                     `json:"channel_id"`     // Channel ID
    ChaincodeId   string                     `json:"chaincode_id"`   // Chaincode ID
    TransactionId string                     `json:"transaction_id"` // Transaction ID
    Timestamp     *google_protobuf.Timestamp `json:"timestamp"`      // Transaction timestamp
    IsInvalid     bool                       `json:"is_invalid"`     // Is transaction invalid
    Payload       interface{}                `json:"payload"`        // Transaction Payload
}
```

One blockchain transaction event sample as follows:

```code
{
    "block_number":63,
    "block_hash":"vTRmfHZ3aaecbbw2A5zPcuzekUC42Lid3w+i6dOU5C0=",
    "channel_id":"pubchain",
    "chaincode_id":"pubchain-c4:",
    "transaction_id":"243eaa6e695cc4ce736e765395a64b8b917ff13a6c6500a11558b5e94e02556a",
    "timestamp":{
        "seconds":1521189855,
        "nanos":192203115
    },
    "is_invalid":false,
    "payload":{
        "id":"did:axn:4debe20b-ca00-49b0-9130-026a1aefcf2d",
        "metadata":{
            "member_id_value":"3714811988020512",
            "member_mobile":"6666",
            "member_name":"8777896121269017",
            "member_truename":"Tony"
        }
    }
}
```
