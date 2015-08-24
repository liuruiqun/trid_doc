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
                "is_first_login": true
            }
            ```


    - 注意事项：
        - `token`为以后与服务端交互所使用的，也就是说如果在`token`有效期内，只需要将其存储起来便可以和服务端交互
        - `huanxin_id`和`huanxin_password`为客户端与环信服务端交互所需要的唯二信息
        - 如果错误码为4，则代表在注册环信用户的时候出错了，此时会讲环信的的reponse附带，eg
        - `is_first_login`若用户首次登录则返回`true`
        
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
                    "pf_answers":[
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
		    "count": 2, 
		    "offset": 0, 
		    "limit": 2, 
		    "pictures": [
		        {
		            "_id": {
		                "$id": "55c97dae2ff2e1f80d000031"
		            }, 
		            "picture": "uploads/18615794931/14392682706855733d.jpeg", 
		            "word": "123", 
		            "createtime": "1439268270"
		        }, 
		        {
		            "_id": {
		                "$id": "55c97e3d2ff2e1f80d000032"
		            }, 
		            "picture": "uploads/18615794931/14392684136967553d.jpeg", 
		            "word": "123", 
		            "createtime": "1439268413", 
		            "created_by": {
		                "$id": "55c95c32ab45d8580c22c224"
		            }
		        }, 
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
			"type":"picture_upload_response",
			"success":true,
			"error_no":0,
			"error_msg":null,
			"picture":"14392797035a676b3d.jpeg"
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
		      "18615794931"
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
		      "18615794931"
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
	    		- response_to 数值对应回复的评论ID。若为0，则表示该回复为初始评论。
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
			            "$id": "55c999ee2ff2e1ac1c00004a"
			        },
			        "picture": "uploads/18615794931/143927550251304d3d.jpeg",
			        "word": "123",
			        "createtime": "1439275502",
			        "created_by": {
			            "$id": "55c95c32ab45d8580c22c224"
			        },
			        "comments": [
			            {
			                "response_to": 0,
			                "content": "hello",
			                "id": 1,
			                "user_id": {
			                    "$id": "55d591d12f2c8214a62fe0a7"
			                },
			                "create_time": 1440122984
			            },
			            {
			                "response_to": 0,
			                "content": "hello",
			                "id": 2,
			                "user_id": {
			                    "$id": "55d591d12f2c8214a62fe0a7"
			                },
			                "create_time": 1440122993
			            },
			            {
			                "response_to": 1,
			                "content": "hello",
			                "id": 3,
			                "user_id": {
			                    "$id": "55d591d12f2c8214a62fe0a7"
			                },
			                "create_time": 1440123020
			            },
			            {
			                "response_to": 2,
			                "content": "hello",
			                "id": 4,
			                "user_id": {
			                    "$id": "55d591d12f2c8214a62fe0a7"
			                },
			                "create_time": 1440123030
			            },
			            {
			                "response_to": 3,
			                "content": "hello",
			                "id": 5,
			                "user_id": {
			                    "$id": "55d591d12f2c8214a62fe0a7"
			                },
			                "create_time": 1440123032
			            }
			        ]
			    }
			}
		```
		- 注意事项
	    		- response_to 数值对应回复的评论ID。若为0，则表示该回复为初始评论。
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



##获取偏好问题
- c->s: 
	- 请求方式 POST
	- URL：http://101.200.89.240/index.php?r=preference/request 
		```
		{
			"type":"pf_question_request",
			"tel":"18615794931",
			"token":"2775495bf4006a925e1540268e083944"
		}
	
		```
	- 注意事项
	    - 无
- s->c:
    - 成功返回：
    	```
    	{
    		"type":"pf_question_response",
    		"success":true,
    		"error_no":0,
    		"error_msg":"",
    		"question":{
    			"pf_id":0,
    			"pic0_enc":"pic0 base64 encoding",
    			"pic1_enc":"pic1 base64 encoding",
                "description":"what is your favourate, tea or coffee?"
    		}
	    }
      		
	   	```
    - 失败返回
		```
        {
            "type:" "pf_question_response",
            "success": false,
            "error_no": 1,
            "error_msg": "json decode failed."
            "question":null
        }
            	
        ```


    - 错误码:
    
        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
	|3|tel not found.|电话号码错误|
	|4|token not valid.|token不正确，可能是过期或者错误了，需要通过登录流程重新获取新的token|
	|5|no available pf questions.|数据库中已经不存在用户还未回答过的偏好问题|
	|6|database error.|数据库错误|
        |7|error in reading files.|无法读取偏好问题所对应的图片和描述，或者读取出错|
        |8|base64 encryption error.|base64加密图片出错|

##上传偏好回答
- c->s: 
    - 请求方式 POST
    - URL：http://101.200.89.240/index.php?r=preference/upload 
        ```
        {
            "type":"pf_answer_upload",
            "tel":"18615794931",
            "token":"2775495bf4006a925e1540268e083944",
            "answer":{
                "pf_id":0,     //对应的偏好问题编号
                "choice"0      //0或者1
            }
        }
    
        ```
    - 注意事项
        - 无
- s->c:
    - 成功返回：
        ```
        {
            "type":"pf_answer_confirm",
            "success":true,
            "error_no":0,
            "error_msg":""
        }
            
        ```
    - 失败返回
        ```
        {
            "type:" "pf_answer_confirm",
            "success": false,
            "error_no": 1,
            "error_msg": "json decode failed."
        }
                
        ```


    - 错误码:
    
        |error_no|error_msg|description|
        |--------|---------|-----------|
        |1|json decode failed.|输入不是有效的json对象|
        |2|input not valid.|请求不完整，缺少某些属性|
        |3|tel not found.|电话号码错误|
        |4|token not valid.|token不正确，可能是过期或者错误了，需要通过登录流程重新获取新的token|
        |5|answer not valid|提交的回答无效，可能pf_id或choice设置正确|
        |6|database error.|数据库错误|
     

	


###个人资料

###设置
####号码更换
- c: 用户号码更换
- c->s: 用户号码更换请求: `picture_like_request`
####接收暗恋信息
