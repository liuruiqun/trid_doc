#user api
##前言

请求输入放在HTTP `POST`/`PUT`/`DELETE`/`PATCH`请求的`body`里面，并且设置header中的`Content-Type: application/json`

例如，`短信验证请求`对应下面的curl命令



	curl \
	    -X POST \
	    -H "Content-Type: application/json" \
	    -d '{"type":"sms_validation_request","tel":"13811112222","time":"1439280893"}' \
	    http://101.200.89.240/index.php?r=user/sms-validation-request

对应下面的HTTP请求报文：

	POST http://101.200.89.240/index.php?r=user/sms-validation-request HTTP/1.1
	Host: 101.200.89.240
	User-Agent: curl/7.43.0
	Accept: */*
	Content-Type: application/json
	Content-Length: 73
	{"type":"sms_validation_request","tel":"13811112222","time":"1439280893"}




##短信验证请求

- c->s
    - 请求方式：POST
    - URL：http://101.200.89.240/index.php?r=user/sms-validation-request
    - 请求格式：

            {
                "type":"sms_validation_request",
                "tel":"13811113333",
                "time":"1439280893"
            }
            
    - 注意事项:
        - 每个客户端/手机号，每60s只能请求一次，请求过快会返回错误
        - 现阶段没有调用第三方短信服务，验证码直接设置为`123456`



- s->c
    - 成功返回：

            {
                "type": "sms_validation_send",
                "success": true,
                "error_no": 0,
                "error_msg": null
            }


    - 失败返回:


            {
                "type:" "sms_validation_send",
                "success": false,
                "error_no": 1,
                "error_msg": "json decode failed."
            }


    - 错误码:
    
        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
        |3|database error.|数据库异常|
        |4|3rd party sms send failed.|第三方短信服务出错，通常是请求频率过高|




##短信验证码验证

- c->s
    - 请求方式：POST
    - URL：http://101.200.89.240/index.php?r=user/sms-validation-code
    - 请求格式：

            {
                "type":"sms_validation_code",
                "tel":"13811113333",
                "time":"1439280893",
                "code":"123456"
            }
            
    - 注意事项:
        - 无


- s->c
    - 成功返回： 客户端应该存储token，

            ```
            {
                "type": "sms_validation_result",
                "success": true,
                "token": "c0gwODVOcmVFS21OakVoQjFiV0JRaXBWYkFLVDZwd2NVaE1HTlBrSDRnWVZaWlA5",
                "huanxin_id": "13811113333",
                "huanxin_pwd": "Q0VsckJ0M0QrdHdwakpTalNxWUVoVHJS",
                "error_no": 0,
                "error_msg": null,
                "basic_info_required": true
            }
            ```


    - 注意事项：
        - `token`为以后与服务端交互所使用的，也就是说如果在`token`有效期内，只需要将其存储起来便可以和服务端交互
        - `huanxin_id`和`huanxin_password`为客户端与环信服务端交互所需要的唯二信息
        - 如果错误码为4，则代表在注册环信用户的时候出错了，此时会讲环信的的reponse附带，eg
        - `basic_info_required`若为`true`, 即表明用户还没有完成基本信息的填写，据此，客户端需要进行9次二选一，并填写出生日期
        
            ```
            {
              "type": "register",
              "success": false,
              "error_no": 4,
              "error_msg": "huanxin error.",
              "huanxin_response": {
                "error": "duplicate_unique_property_exists",
                "timestamp": 1439968399454,
                "duration": 0,
                "exception": "org.apache.usergrid.persistence.exceptions.DuplicateUniquePropertyExistsException",
                "error_description": "Application 0c6fc7c0-4584-11e5-ad97-bf184b66b711 Entity user requires that property named username be unique, value of test5 exists"
              }
            }
            ```

    - 失败返回:

            {
                "type:" "sms_validation_result",
                "success": false,
                "error_no": 1,
                "error_msg": "json decode failed."
            }

    - 错误码:
    
    
        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
        |3|validation code invalid.|验证码错误|
        |4|huanxin error.|环信操作时出错|
        |5|database error.|数据库错误|


##查找用户

- c->s
    - 请求方式：POST
    - URL：http://101.200.89.240/index.php?r=user/search
    - 请求格式：
        - 按照昵称查找
            ```
            {
                "type":"search",
                "token":"YSs2SGhPaXZUbm1UMmV1aGJkVVRiMXVFZUIvQ2lLREg0bFdpWHhOQlJsRUpjeDlY",
                "tel":"13811112222",
                "search":
                {
                    "username":"test"
                }
            }
            ```

        - 按照手机号查找

            ```
            {
                "type":"search",
                "token":"YSs2SGhPaXZUbm1UMmV1aGJkVVRiMXVFZUIvQ2lLREg0bFdpWHhOQlJsRUpjeDlY",
                "tel":"13811112222",
                "search":
                {
                    "tel":"13811113333"
                }
            }
            ```

        - 按照`_id`查找

            ```
            {
                "type":"search",
                "token":"YSs2SGhPaXZUbm1UMmV1aGJkVVRiMXVFZUIvQ2lLREg0bFdpWHhOQlJsRUpjeDlY",
                "tel":"13811112222",
                "search":
                {
                    "_id":"55d591d12f2c8214a62fe0a7"
                }
            }
            ```

    - 注意事项:
        - 无


- s->c
    - 成功返回：
    
            ```
            {
              "type": "search_result",
              "success": true,
              "error_no": 0,
              "error_msg": null,
              "count": 2,
              "offset": 0,
              "limit": 2,
              "users": [
                {
                  "_id": {
                    "$id": "55c95c32ab45d8580c22c224"
                  },
                  "tel": "18615794931",
                  "username": "test"
                },
                {
                  "_id": {
                    "$id": "55c9af7b924029a5f6bb09e1"
                  },
                  "tel": "13811112222",
                  "username": "test"
                }
              ]
            }
            ```
    

    - 注意事项：
        - 无

    - 失败返回:

            {
                "type:" "search_result"
                "success": false
                "error_no": 1
                "error_msg": "json decode failed."
            }

    - 错误码:
    
    
        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
        |3|tel not found.|电话号码错误|
        |5|token not valid.|token不正确|
        |6|tel not verified.|电话号码未验证|
        |7|database error.|数据库错误|

##获取自己的设置

- c->s
    - 请求方式：POST
    - URL：http://101.200.89.240/index.php?r=user/profile
    - 请求格式：


            {
                "type":"profile",
                "token":"YSs2SGhPaXZUbm1UMmV1aGJkVVRiMXVFZUIvQ2lLREg0bFdpWHhOQlJsRUpjeDlY",
                "tel":"13811113333",
            }

            
    - 注意事项:
        - 无


- s->c
    - 成功返回：
    

            {
                "success":true,
                "error_no":0,
                "error_msg":null,
                "profile":{
                    "_id":{
                        "$id":"55d591d12f2c8214a62fe0a7"
                    },
                    "username":null,
                    "huanxin_id":"13811113333",
                    "huanxin_password ":"Q0VsckJ0M0QrdHdwakpTalNxWUVoVHJS",
                    "pf_answer":[
                        {
                            "pf_id": 0,
                            "choice": 1
                        },
                        {
                            "pf_id": 1,
                            "choice": 1
                        }
                    ]
                }
            }


    - 注意事项：
        - 无

    - 失败返回:

            {
                "type:" "profile"
                "success": false
                "error_no": 1
                "error_msg": "json decode failed."
            }

    - 错误码:
    
    
        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
        |3|tel not found.|电话号码错误|
        |4|token not valid.|token不正确|


#picture api(秀爱社区)
##查看社区信息
- c->s: 
	- 请求方式 POST
	- URL：http://101.200.89.240/index.php?r=picture/search
	
		```
		{
			"type":"picture_search_request",
			"tel":"18782945332",
			"token":"18782945332"
		}
		```

- s->c: 
	- 若请求成功，返回

		```
		{
		  "type": "picture_search_response",
		  "success": true,
		  "error_no": 0,
		  "error_msg": null,
		  "count": 17,
		  "offset": 0,
		  "limit": 17,
		  "pictures": [
		    {
		      "_id": {
		        "$id": "55e05d1c3b505f10667f9239"
		      },
		      "picture": "/uploads/18615194931/1440767260YTZvRmpO.jpeg",
		      "word": "大家好!",
		      "like": 0,
		      "like_by": [],
		      "comments": [],
		      "createtime": 1440767260,
		      "created_by": {
		        "$id": "55c4c37b211c85467bdcef52"
		      }
		    },
		    {
		      "_id": {
		        "$id": "55e15d653b505f464a7f9288"
		      },
		      "picture": "/uploads/18615194931/1440832869cTFrV28r.jpeg",
		      "word": "hello!",
		      "like": 1,
		      "like_by": [
		        "test"
		      ],
		      "comments": [
		        {
		          "response_to": 2,
		          "content": "hello",
		          "id": 1,
		          "nick": "test",
		          "create_time": 1440838697
		        }
		      ],
		      "createtime": 1440832869,
		      "created_by": {
		        "$id": "55c4c37b211c85467bdcef52"
		      }
		    }
		  ]
}
		```
		- 注意事项
	    		- picture 返回的是图片存放的相对地址，通过http请求即可获取，如
	    		- http://101.200.89.240/uploads/18615794931/14392684136967553d.jpeg。
	- 若请求失败，返回
	
		```
		{
			"type":"picture_search_response",
			"success":false,
			"error_no":1,
			"error_msg":
			"json decode failed."
		}
		```
	- 错误码:

        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
        |3|tel not found.|电话号码错误|
        |4|token not valid.|token不正确，可能是过期或者错误了，需要通过登录流程重新获取新的token|
        |5|tel not verified.|电话号码未通过短信验证|
        |6|database error.|数据库错误|
	


##用户上传图片及文字
- c->s: 
	- 请求方式 POST
	- URL：http://101.200.89.240/index.php?r=picture/upload
	
		```
		{
			"type":"picture_upload_request",
			"token":"18782945332",
			"tel":"18782945332",
			"words":"hello!",
			"picture":"data:image\/jpeg;base64..."
		
		}
		```

- s->c： 
	- 若请求成功，则返回

		```
		{
			  "type": "picture_upload_response",
			  "success": true,
			  "error_no": 0,
			  "error_msg": null,
			  "picture": "/uploads/18615194931/1440764869ZnJKemV6.jpeg"
		}
		```
		
	- 若请求失败，返回
	
		```
		{
			"type":"picture_search_response",
			"success":false,
			"error_no":1,
			"error_msg":
			"json decode failed."
		}
		```
		
	- 错误码:
    
        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
        |3|tel not found.|电话号码错误|
        |4|token not valid.|token不正确，可能是过期或者错误了，需要通过登录流程重新获取新的token|
        |5|tel not verified.|电话号码未通过短信验证|
        |6|database error.|数据库错误|

##撤销已发送图片信息

- c->s: 
	- 请求方式 POST
	- URL：http://101.200.89.240/index.php?r=picture/delete 
		```
		{
			"type":"picture_delete_request",
			"token":"2775495bf4006a925e1540268e083944",
			"tel":"18615794931",
			"picture_id":"55caecb43b505f7b698b4567"
		}
	
		```
- s->c： 
	- 若请求成功，则返回

		```
		{
		  	"type" : "picture_delete_response",
            		"success" : true,
            		"error_no" : 0,
            		"error_msg" : null,
		}
		```
		
	- 若请求失败，返回
	
		```
		{
			"type":"picture_like_response",
			"success":false,
			"error_no":1,
			"error_msg":
			"json decode failed."
		}
		```
		
	- 错误码:
    
        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
        |3|tel not found.|电话号码错误|
        |4|token not valid.|token不正确，可能是过期或者错误了，需要通过登录流程重新获取新的token|
        |5|query result is null.|数据库返回NULL|
        |6|database error.|数据库错误|
	    |7|permission denied.|没有操作权限|

##点赞

- c->s: 
	- 请求方式 POST
	- URL：http://101.200.89.240/index.php?r=picture/like 
		```
		{
			"type":"picture_like_request",
			"token":"2775495bf4006a925e1540268e083944",
			"tel":"18615794931",
			"picture_id":"55caecb43b505f7b698b4567"
		}
	
		```
- s->c： 
	- 若请求成功，则返回

		```
		{
		  "type": "picture_like_response",
		  "success": true,
		  "error_no": 0,
		  "error_msg": null,
		  "picture": {
		    "_id": {
		      "$id": "55caecb43b505f7b698b4567"
		    },
		    "picture": "uploads/18615794931/1439362228354a673d.jpeg",
		    "word": "hello!",
		    "like": 1,
		    "like_by": [
		      "user1","user2"
		    ],
		    "createtime": 1439362228,
		    "created_by": {
		      "$id": "55c4c339211c85467bdcef51"
		    }
		  }
		}
		```
		
	- 若请求失败，返回
	
		```
		{
			"type":"picture_like_response",
			"success":false,
			"error_no":1,
			"error_msg":
			"json decode failed."
		}
		```
		
	- 错误码:
    
        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
        |3|tel not found.|电话号码错误|
        |4|token not valid.|token不正确，可能是过期或者错误了，需要通过登录流程重新获取新的token|
        |5|tel not verified.|电话号码未通过短信验证|
        |6|database error.|数据库错误|

##取消点赞

- c->s: 
	- 请求方式 POST
	- URL：http://101.200.89.240/index.php?r=picture/unlike 
		```
		{
			"type":"picture_unlike_request",
			"token":"2775495bf4006a925e1540268e083944",
			"tel":"18615794931",
			"picture_id":"55caecb43b505f7b698b4567"
		}
	
		```
- s->c： 
	- 若请求成功，则返回

		```
		{
		  "type": "picture_unlike_response",
		  "success": true,
		  "error_no": 0,
		  "error_msg": null,
		  "picture": {
		    "_id": {
		      "$id": "55caecb43b505f7b698b4567"
		    },
		    "picture": "uploads/18615794931/1439362228354a673d.jpeg",
		    "word": "hello!",
		    "like": 1,
		    "like_by": [
		      "user1"
		    ],
		    "createtime": 1439362228,
		    "created_by": {
		      "$id": "55c4c339211c85467bdcef51"
		    }
		  }
		}
		```
		
	- 若请求失败，返回
	
		```
		{
			"type":"picture_unlike_response",
			"success":false,
			"error_no":1,
			"error_msg":
			"json decode failed."
		}
		```
		
	- 错误码:
    
        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
        |3|tel not found.|电话号码错误|
        |4|token not valid.|token不正确，可能是过期或者错误了，需要通过登录流程重新获取新的token|
        |5|picture_id not valid.|图片不存在|
        |6|permission denied.|没有操作权限|

##评论

- c->s: 
	- 请求方式 POST
	- URL：http://101.200.89.240/index.php?r=picture/comment 
		```
		{
			"type":"picture_comment_request",
			"token":"2775495bf4006a925e1540268e083944",
			"tel":"18615794931",
			"picture_id":"55caecb43b505f7b698b4567"
			"comment":{
				"response_to":2,
				"content":"hello"
			}
		}
	
		```
		- 注意事项
	    		- response_to 数值对应回复的评论ID。若为1，则表示该回复为初始评论。
	    		- 每一条回复都有唯一的ID，从1开始递增。
- s->c： 
	- 若请求成功，则返回

		```
		{
			  "type": "picture_comment_response",
			  "success": true,
			  "error_no": 0,
			  "error_msg": null,
			  "picture": {
			    "_id": {
			      "$id": "55e15d653b505f464a7f9288"
			    },
			    "picture": "/uploads/18615194931/1440832869cTFrV28r.jpeg",
			    "word": "hello!",
			    "like": 1,
			    "like_by": [
			      "test"
			    ],
			    "comments": [
			      {
			        "response_to": 2,
			        "content": "hello",
			        "id": 1,
			        "nick": "test",
			        "create_time": 1440838697
			      }
			    ],
			    "createtime": 1440832869,
			    "created_by": {
			      "$id": "55c4c37b211c85467bdcef52"
			    }
			  }
		}
		```
		- 注意事项
	    		- response_to 数值对应回复的评论ID。若为1，则表示该回复为初始评论。
	    		- 每一条回复都有唯一的ID，从1开始递增。
		
	- 若请求失败，返回
	
		```
		{
			"type":"picture_like_response",
			"success":false,
			"error_no":1,
			"error_msg":
			"json decode failed."
		}
		```
		
	- 错误码:
    
        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
        |3|tel not found.|电话号码错误|
        |4|token not valid.|token不正确，可能是过期或者错误了，需要通过登录流程重新获取新的token|
        |5|tel not verified.|电话号码未通过短信验证|
        |6|database error.|数据库错误|


#用户信息相关API

#上传基本信息
- 说明
	- 用户登陆时，若返回的`sms_validation_result`消息中`basic_info_required`字段为`true`,则表明用户还没有完成基本信息的收集，客户端需要进行9次二选一，并填写出生日期。其中，性别和出生日期包含在`basic_info_upload`消息中上传到服务器，而其他的二选一通过`pf_answer_upload`上传。

- c->s
	- 请求方式 POST
	- URL http://101.200.89.240/index.php?r=info/basic-info-upload
		
		```
		{
    		"type":"basic_info_upload",
    		"tel":"13455556666",
    		"token":"RWdRNEhPN2E1cGJXOXZNQktyd0tJakxYVjNOZHQyMkQwdE43cDMxZk5MNHloOWwx",
    		"sex":0,
    		"birthdate":{ 
    			"month":2, 
    			"day":29, 
    			"year":2000
    		}
		}
		```

	- 注意事项
		- `sex`字段只能取值为0或1, 0表明性别为male, 1表明性别为female
		- 服务器会对`birthdate`的有效性进行检查，若非法，返回错误码2

- s->c
	- 成功返回

		```
		{
			"type":"basic_info_upload_result",
			"success":true,
			"error_no":0,
			"error_msg":"",
		}
		```

	- 失败返回

		```
		{
			"type":"basic_info_upload_result",
			"success":true,
			"error_no":0,
			"error_msg":"",
		}
		```

	- 错误码
	
		|error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性，或者某些属性格式不对|
        |3|tel not found.|电话号码错误|
        |4|token not valid.|token不正确，可能是过期或者错误了，需要通过登录流程重新获取新的token|
        |5|database error.|数据库错误|

##获取偏好图片
- 说明
	- 此API用于从服务器获取偏好图片，服务器返回的偏好图片对应的`pf_id`是随机的
- c->s
	- 请求方式 POST
	- URL http://101.200.89.240/index.php?r=info/pf-picture-request

		```
		{
    		"type":"pf_picture_request",
    		"tel":"13455556666",
    		"token":"d2VqYlU1bWZtWGdaaHFzaSsrcTg0ZXdMQkRJdWgwWjdUQnA0L1NyN3RSNkpOaWJ3"
		}
		```

	- 注意事项
	    - 无

- s->c
    - 成功返回

    	```
    	pf_picture_result
		{
			"type":"pf_picture_result",
			"success":true,
			"error_no":0,
			"error_msg":"",
			"pf":{
				"pf_id":0,
				"pic0":{
					"type":"jpg",
					"name_en":"coffee",
					"name_cn":"\u5496\u5561",
					"data":"BASE64 encoding data"
				},
				"pic1":{
					"type":"jpg",
					"name_en":"tea",
					"name_cn":"\u8336",
					"data":"BASE64 encodeing data"
				}
			}
		}		
	   	```

	- 注意事项
		- `name_cn`字段对应图片内容的中文名，其值为相应汉字的UTF编码
		- `data`字段对应图片的base64编码

    - 失败返回

		```
		{
			"type":"pf_picture_result",
			"success":true,
			"error_no":1,
			"error_msg":"json decode failed."
		}     	
        ```


    - 错误码:
    
        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
	    |3|tel not found.|电话号码错误|
	    |4|token not valid.|token不正确，可能是过期或者错误了，需要通过登录流程重新获取新的token|
	    |5|no available pf picture.|数据库中已经不存在用户还未回答过的二选一偏好|
	    |6|database error.|数据库错误|
        |7|error in reading image files or unknown image type.|无法读取二选一偏好对应的图片，或图片格式未知|
        |8|base64 encryption error.|base64加密图片出错|

##偏好回答上传
- c->s: 
    - 请求方式 POST
    - URL http://101.200.89.240/index.php?r=info/pf-answer-upload

        ```
        {
    		"type":"pf_answer_upload",
    		"tel":"13455556666",
    		"token":"d2VqYlU1bWZtWGdaaHFzaSsrcTg0ZXdMQkRJdWgwWjdUQnA0L1NyN3RSNkpOaWJ3",
    		"count":2,
    		"pf_answer":[
    			{
    				"pf_id":1,
    				"choice":1
    			},
    			{
    				"pf_id":0, 
    				"choice":0
    			}
    		]    
		}
        ```

    - 注意事项
    	- `pf_answer`必须为json数组，即使只有一个元素
        - `count`对应`pf_answer`中的元素数量，且必须相等，否则返回错误码2
        - `pf_answer`中各元素的`pf_id`不能相同，否则返回错误码2
        - 若`pf_id`和`choice`无效，返回错误码2，其中`choice`取值为0或1，表示相应的二选一答案

- s->c:
    - 成功返回

        ```
        {
            "type":"pf_answer_upload_result",
            "success":true,
            "error_no":0,
            "error_msg":""
        }            
        ```

    - 失败返回

        ```
        {
            "type:" "pf_answer_upload_result",
            "success": false,
            "error_no": 1,
            "error_msg": "json decode failed."
        }
                
        ```

    - 错误码:
    
        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性，或属性值错误|
        |3|tel not found.|电话号码错误|
        |4|token not valid.|token不正确，可能是过期或者错误了，需要通过登录流程重新获取新的token|
        |5|database error.|数据库错误|


#好友管理API

##添加好友
- 说明
    - 好友关系具有对称性，即A是B的好友，则B一定是A的好友
    - 好友关系不具有反身性，即自己和自己不是好友关系
    - 添加好友成功后，服务器会通过环信以透传消息的方式通知对方
    - 除了发送`add_friend_request`外，`accept_invitation_request`也可能会向好友列表添加成员

- c->s:
    - 请求方式 POST
    - URL http://101.200.89.240/index.php?r=contact/add-friend

        ```
        {
            "type": "add_friend_request",
            "tel": "13811112222",
            "token"："NDFOcXI4V04vdDNRckNJT3UrOFFTaGF5OUpHdld5aUFBUVkyRTEwQXZmSmhGd05z",
            "peer_tel": "15677778888",
            "word": "hey, I've crushed on you for a long time."
        }
        ```

    - 注意事项
        - `word`字段为可选项

- s->c:
    - 成功返回

        ```
        {
            "type": "add_friend_result",
            "success"：true,
            "error_no": 0,
            "error_msg": "",
            "friend": {
                "peer_tel":"15677778888",
                "huanxin_id":"15677778888",
                "type":0,
                "chat_title":"aojiao77254",
                "expire":1440681544
            }
            "word":"hey, I've crushed on you for a long time."
        }
        ```
        - 注意事项
            - `type`字段表示好友类型，0表示己方暗恋对方，1表示己方被对方暗恋，2表示陌生人
            - `expire`字段表示对话过期时间
            - 好友添加成功后，服务器调用环信接口向对方发送**透传消息**，该透传消息结构如下

            	```
            	{
					"target_type":"users",
					"target":["15677778888"], //对方环信id 
					"msg":{
						"type":"cmd", 
						"action":"new_friend_notification"
					},
					"from":"admin",  //由服务器发送
					"ext":{
						"friend":{
                			"peer_tel":"13811112222",
                			"huanxin_id":"13811112222",
                			"type":1,
                			"chat_title":"aojiao77254",
                			"expire":1440681544
           				 },
						"word":"hey, I've crushed on you for a long time."
					}
				}
            	```

    - 失败返回

        ```
        {
            "type": "add_friend_result",
            "success": false,
            "error_no": 8,
            "error_msg": "peer user reject."
        }
        ```

    - 错误码

        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
        |3|tel not found.|电话号码错误|
        |4|token not valid.|token不正确，可能是过期或者错误了，需要通过登录流程重新获取新的token|
        |5|peer_tel can't be same as tel.|不能添加自己为好友|
        |6|peer user not exist.|对方还不是三天用户，此时己方可请求服务器向对方发送邀请|
        |7|peer user is already in the friend list.|对方已经是己方好友|
        |8|peer user reject.|对方拒绝添加好友|
        |9|fail to notify peer user.|给对方的好友添加通知发送失败|


## 删除好友
- 说明
    - 删除好友时，双方`friend_list`中的相关记录均会被删除，以保证好友关系的对称性
    - 删除好友成功后，服务器会通知对方，但目前该功能还在实现中
- c->s
    - 请求方式 POST
    - URL http://101.200.89.240/index.php?r=contact/delete-friend

        ```
        {
            "type":"delete_friend_request",
            "tel":"13455556666",
            "token":"NDFOcXI4V04vdDNRckNJT3UrOFFTaGF5OUpHdld5aUFBUVkyRTEwQXZmSmhGd05z",
            "peer_tel":"13788889999"
        }
        ```
- s->c
    - 成功返回

        ```
        {
            "type": "delete_friend_result",
            "success": true,
            "error_no": 0,
            "error_msg": ""
        }
        ```

    - 失败返回

        ```
        {
            "type": "delete_friend_result",
            "success": false,
            "error_no": 1,
            "error_msg": "json decode failed."
        }
        ```

    - 错误码

        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
        |3|tel not found.|电话号码错误|
        |4|token not valid.|token不正确，可能是过期或者错误了，需要通过登录流程重新获取新的token|
        |5|friend don't exist.|好友不存在|

## 获取好友列表
- c->s
    - 请求方式 POST
    - URL http://101.200.89.240/index.php?r=contact/get-friend-list

        ```
        {
            "type":"get_friend_list",
            "tel":"13455556666",
            "token":"NDFOcXI4V04vdDNRckNJT3UrOFFTaGF5OUpHdld5aUFBUVkyRTEwQXZmSmhGd05z"
        }
        ```
- s->c
    - 成功返回

        ```
        {
            "type": "friend_list",
            "success": true,
            "error_no": 0,
            "error_msg": "",
            "count":3,
            "friend_list":[
                {
                    "peer_tel":"15677778888",
                    "huanxin_id":"15677778888",
                    "type":0,
                    "chat_title":"aojiao77254",
                    "expire":1440681544
                },
                ...
            ]
        }
        ```

    - 失败返回

        ```
        {
            "type": "friend_list",
            "success": false,
            "error_no": 1,
            "error_msg": "json decode failed."
        }
        ```

    - 错误码

        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
        |3|tel not found.|电话号码错误|
        |4|token not valid.|token不正确，可能是过期或者错误了，需要通过登录流程重新获取新的token|

## 发送用户邀请
- 说明
    - 当用户添加好友时，若返回的结果为对方用户不存在，此时客户端可请求发送用户邀请。
    
- c->s
    - 请求方式 POST
    - URL http://101.200.89.240/index.php?r=contact/send-invitation

        ```
        {
            "type":"send_invitation_request",
            "tel":"13455556666",
            "token":"NDFOcXI4V04vdDNRckNJT3UrOFFTaGF5OUpHdld5aUFBUVkyRTEwQXZmSmhGd05z",
            "peer_tel":"13788889999",
            "word":"hello!"
        }
        ```

    - 注意事项
        - `word`字段可选
        - 每个客户端/手机号，每60s只能请求一次，请求过快会返回错误
        - 在向某个手机号发送用户邀请后，24小时内，若再次向同一手机号发送用户邀请，则返回错误

- s->c
    - 成功返回

        ```
        {
            "type": "send_invitation_result",
            "success": true,
            "error_no": 0,
            "error_msg": "",
            "invitation_send": {
                "peer_tel": "15677778888",
                "num":1,  //向对方发送用户邀请的累计次数
                "expire":
                "word":""
            }
        }
        ```

    - 失败返回

        ```
        {
            "type": "send_invitation_result",
            "success": true,
            "error_no": 1,
            "error_msg": "json decode failed."   
        }
        ```

    - 错误码

        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
        |3|tel not found.|电话号码错误|
        |4|token not valid.|token不正确，可能是过期或者错误了，需要通过登录流程重新获取新的token|
        |5|peer tel can't be same as tel.|不能向自己发送用户邀请|
        |6|peer user already exist.|对方用户已经存在|
        |7|last invitation not expire.|上一次向对方手机发送的用户邀请还未过期|
        |8|fail to send invitation sms.|发送用户邀请短信失败，原因可能是客户端请求过快|


## 删除发送的用户邀请记录

- 说明
    - 该请求只是将数据库中invitation_send_list的相关记录删除，除此之外，无任何其他作用。

- c->s
    - 请求方式 POST
    - URL http://101.200.89.240/index.php?r=contact/delete-invitation-send

        ```
        {
            "type":"delete_invitation_send_request",
            "tel":"13455556666",
            "token":"NDFOcXI4V04vdDNRckNJT3UrOFFTaGF5OUpHdld5aUFBUVkyRTEwQXZmSmhGd05z",
            "peer_tel":"13233334444"
        }
        ```

- s->c
    - 成功返回

        ```
        {
            type: "delete_invitation_send_result",
            success: true,
            error_no: 0,
            error_msg: ""
        }
        ```

    - 失败返回

        ```
        {
            type: "delete_invitation_send_result",
            success: true,
            error_no: 1,
            error_msg: "json decode failed."
        }
        ```

    - 错误码
    
        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
        |3|tel not found.|电话号码错误|
        |4|token not valid.|token不正确，可能是过期或者错误了，需要通过登录流程重新获取新的token|
        |5|invitation not exist.|邀请记录不存在|


## 获取发送的邀请记录表
- c->s
    - 请求方式 POST
    - URL http://101.200.89.240/index.php?r=contact/get-invitation-send-list

        ```
        {
            "type":"get_invitation_send_list",
            "tel":"13455556666",
            "token":"NDFOcXI4V04vdDNRckNJT3UrOFFTaGF5OUpHdld5aUFBUVkyRTEwQXZmSmhGd05z"
        }
        ```

- s->c
    - 成功返回

        ```
        {
            "type":"invitation_send_list",
            "success":true,
            "error_no":0,
            "error_msg":"",
            "count":2,
            "invitation_send_list": [
                {
                    "peer_tel":"15677778888",
                    "num":1,
                    "expire":464131646464,
                    "word":"hello"
                },
                ...
            ]
        }
        ```

    - 失败返回

        ```
        {
            "type":"invitation_send_list",
            "success":false,
            "error_no":1,
            "error_msg":"json decode failed."
        }
        ```

    - 错误码

        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
        |3|tel not found.|电话号码错误|
        |4|token not valid.|token不正确，可能是过期或者错误了，需要通过登录流程重新获取新的token|


## 获取收到的邀请记录表
- 说明
    - 用户当且仅当是第一次登录时，才需要发送`get_invitation_recv_list`以查询是否有用户邀请过自己
    - 根据`get_invitation_recv_list`返回的结果，客户端判断是否需要发送`accept_invitation_request`

- c->s
    - 请求方式 POST
    - URL http://101.200.89.240/index.php?r=contact/get-invitation-recv-list

        ```
        {
            "type":"get_invitation_recv_list",
            "tel":"13233334444",
            "token":"SEZUVFB3NThxUFZLVWh2QldIUzJHTklxTVFHM01sTFJtVVNocXZ5MThubko1c3JT"
        }
        ```

- s->c
    - 成功返回

        ```
        {
            "type": "invitation_recv_list",
            "success": true,
            "error_no": 0,
            "error_msg": "",
            "count": 2,
            "invitation_recv_list":[
                {
                    "peer_tel": "15677778888",
                    "num":2,
                    "expire":45646965,
                    "word":"hello"
                },
                 ...
            ]
        }
        ```

    - 失败返回

        ```
        type: invitation_recv_list
        {
            "type":"invitation_recv_list",
            "success": false,
            "error_no": 1,
            "error_msg": "json decode failed."
        }
        ```

    - 错误码

        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
        |3|tel not found.|电话号码错误|
        |4|token not valid.|token不正确，可能是过期或者错误了，需要通过登录流程重新获取新的token|



## 接受收到的邀请
- 说明
    - 发送`accept_invitation_request`后，如果无错误发生，则系统自动添加双方为好友
    - 接受邀请成功后，服务器会通知对方邀请成功，但该功能目前还在实现中

- c->s
    - 请求方式 POST
    - URL http://101.200.89.240/index.php?r=contact/accept-invitation

        ```
        {
            "type":"accept_invitation_request",
            "tel":"13233334444",
            "token":"SEZUVFB3NThxUFZLVWh2QldIUzJHTklxTVFHM01sTFJtVVNocXZ5MThubko1c3JT",
            "peer_tel":"13455556666"
        }
        ```

- s->c
    - 成功返回

        ```
        {
            "type": "accept_invitation_result",
            "success": true,
            "error_no": 0,
            "error_msg": "",
            "friend":{
                "peer_tel":"15677778888",
                "huanxin_id":"15677778888",
                "type":0,
                "chat_title":"aojiao0250",
                "expire":782241
            },
            "word":"hello"
        }
        ```

    - 失败返回

        ```
        {
            "type":"accept_invitation_result",
            "success":false,
            "error_no":1,
            "error_msg": "json decode failed"
        }
        ```

    - 错误码

        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
        |3|tel not found.|电话号码错误|
        |4|token not valid.|token不正确，可能是过期或者错误了，需要通过登录流程重新获取新的token|
        |5|peer user not exist.|对方用户不存在，可能的原因是对方用户在发送邀请后，注销了账户|
        |6|invitation not valid.|邀请不存在，或已过期|
        |7|peer user is already in the friend list|和对方用户已经是好友关系|


## 删除接收到的邀请
- 说明
    - 该请求仅删除invitation_recv_list的相关记录，除此之外，无其它任何作用

- c->s
    - 请求方式 POST
    - URL http://101.200.89.240/index.php?r=contact/delete-invitation-recv

        ```
        {
            "type":"delete_invitation_recv_request",
            "tel":"13122223333",
            "token":"VkIwUU5YOTlNZ1ZTK3htWVJZSkd2dkNKY3UxYnlYMWF2SE5mMEtYZmgvZElmeHZu",
            "peer_tel":"13455556666"
        }
        ```

- s->c
    - 成功返回

        ```
        {
            "type":"delete_invitation_recv_result",
            "success": true,
            "error_no": 0,
            "error_msg": ""
        }
        ```

    - 失败返回

        ```
        {
            "type":"delete_invitation_recv_result",
            "success": true,
            "error_no": 1,
            "error_msg": "json decode failed."
        }
        ```

    - 错误码

        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
        |3|tel not found.|电话号码错误|
        |4|token not valid.|token不正确，可能是过期或者错误了，需要通过登录流程重新获取新的token|
        |5|invitation not exist.|请求删除的邀请不存在|

	

###个人资料

###设置
####号码更换
- c: 用户号码更换
- c->s: 用户号码更换请求: `picture_like_request`
####接收暗恋信息
