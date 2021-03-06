# 绅士之庭API

更新日期：2019/3/18

**注意**：目前的API为测试版本，如有问题请向副站汇报。

## 用户相关

### 用户登录

* **/api/account/login**

* **Method:** `POST`
  
*  **Data Params**

   GmGard.Models.LoginModel
   ```
   {
     userName: "username"
     password: "hunter2"
	 rememberMe: true
	 captcha: "16"
   }
   ```

* **Success Response:**

  * **Code:** 200 <br />
    **Content:**  ` { success: true, require2fa: false }  `

  请求成功时，请保存响应中的 `.AspNetCore.Identity.Application` 身份识别COOKIE。
  如果`require2fa`为`true`，则需要额外进行两步验证。请参见两步验证API。
 
* **Error Response:**

  此API返回 HTTP 200 并使用`errors`通知错误信息。

  * `{ success = false, errors: ["提供的用户名或密码不正确"]}`

* **Sample Call:**

  ```
  $.ajax({
	url: "/api/account/login",
	contentType: "application/json",
	data: { "password": "a", userName: "b", captcha: "42" },
	type: "POST",
  });
  ```

* **Notes:**

  * **注意：** 请通过 `/Captcha/CaptchaImage` 获取一个验证图片。请注意需要对应的`.AspNetCore.Session` Cookie.
  
  以下请求默认情况下都需要有 `.AspNetCore.Identity.Application`和`.AspNetCore.Session` COOKIE在请求中。

### 两步验证


* **/api/account/twoFactorAuth**

* **Method:** `POST`
  
*  **Data Params**

   GmGard.Models.Login2FaModel
   ```
   {
     twoFactorCode: "123456"
     rememberMachine: true
	 rememberMe: true
   }
   ```

* **Success Response:**

  * **Code:** 200 <br />
    **Content:**  ` { success: true }  `

  请求成功时，请保存响应中的 `.AspNetCore.Identity.Application` 身份识别COOKIE。
 
* **Error Response:**

  此API返回 HTTP 200 并使用`errors`通知错误信息。

  * `{ success = false, errors: ["秘钥错误"]}`

* **Notes:**

  此API需要先进行正常登陆后，提示`require2fa`后再进行调用。

### 两步验证（应急密码）


* **/api/account/recoveryCode**

* **Method:** `POST`
  
*  **Data Params**

   GmGard.Models.LoginWithRecoveryCodeModel
   ```
   {
     recoveryCode: "123456"
   }
   ```

* **Success Response:**

  * **Code:** 200 <br />
    **Content:**  ` { success: true }  `

  请求成功时，请保存响应中的 `.AspNetCore.Identity.Application` 身份识别COOKIE。
 
* **Error Response:**

  此API返回 HTTP 200 并使用`errors`通知错误信息。

  * `{ success = false, errors: ["无效的应急密码。"]}`

* **Notes:**

  此API需要先进行正常登陆后，提示`require2fa`后再进行调用。应急密码登陆时，cookie有效期只限定本次会话。

### 用户登出

* **/api/account/logOff**

* **Method:** `POST`

* **Success Response:**

  * **Code:** 200

* **Notes:**

  本请求会清除`.AspNetCore.Identity.Application` COOKIE并使先前的COOKIE值无效化。

### 用户详情

* **/api/account/getUser**

* **Method:** `GET`

*  **URL Params**

   **Optional:** `username=[string]`

* **Success Response:**

  * **Code:** 200 <br />
    **Content:**  
	``` 
	{ 
	  userId: 123
	  userName: "用户名"
	  nickName: "[称号] 昵称"
	  comment: "签名"
	  points: 456  // 棒棒糖
	  experience: 789  // 绅士度
	  level: 3
	  avatar: "头像url"
	  roles: ["Writers", "Moderator", "Auditor"]
	}  
	```
	
* **Error Response:**

  * **Code:** `401 UNAUTHORIZED` <br />
    如果未提供username参数，则需要登录才可查看当前用户信息。

* **Notes:**

  只能获取当前用户的信息。用户可能的权限包括："Writers"作者组，"Moderator"管理员，"Auditor"审核组"。

### 用户签到

* **/api/punchIn/do**

* **Method:** `POST`

*  **URL Params**

   **Optional:** `date=[datetime]`

* **Success Response:**

  * **Code:** 200 <br />
    **Content:**  
	``` 
	{ 
	  success: true
      consecutiveDays: 10,
      exp: 5
	}  
	```
	
* **Error Response:**

  * **Code:** `400 BAD REQUEST` <br />
    当签到日期为未来或过早时，将提示“日期无效”。补签时若棒棒糖不足会提示“棒棒糖不足”。补签时若当日已签则提示“此日已签”。

 * **Notes:**

  签到日期时间戳最早记录时间为2019年1月25日。不可补签此日以前的日期。补签需要100棒棒糖。非补签的签到成功可获得1-5经验和棒棒糖。

### 用户签到历史

* **/api/punchIn/history**

* **Method:** `GET`

*  **URL Params**

   **Optional:** `year=[int]&month=[int]`

* **Success Response:**

  * **Code:** 200 <br />
    **Content:**  
	``` 
	[ 
	  { 
        timeStamp: "2019-03-18T09:55:49.243Z",
        isMakeup: false
      }
	]  
	```
	
 * **Notes:**

  签到日期时间戳最早记录时间为2019年1月25日。无法查看此日已前的签到。每次返回的签到数据为指定的年和月的数据。2019年3月9日前的时间戳因失误没有记录时间部分。


## 资源相关

### 资源详情

* **/api/blog/details**

* **Method:** `GET`
  
*  **URL Params**

   **Required:** `id=[non-negative integer]`

* **Success Response:**
  
  GmGard.Models.App.BlogDetails

  * **Code:** 200 <br />
    **Content:** 
    ```
    {
        author: { 
            userId: 123
            userName: "用户名"
            nickName: "昵称"
            comment: "签名"
            points: 456  // 棒棒糖
            experience: 789  // 绅士度
            level: 3
            avatar: "头像url"
        }
        authorDesc: "用户签名"
        categoryId: 21  //分类ID
        parentCategoryId: 4 //父分类ID
        brief: "正文预览"
        content: "<p>正文</p>
        createDate: "2017-10-26T01:49:14.16"
        id: 72850
        imageUrls: ["//static.gmgard.us/Images/upload/14525260149112764.jpg"]
        links: [{name: "链接名称", url: "链接地址", pass: "提取码"}]
        isApproved: true // 是否通过审核，null为待审核
        lockTags: false // 是否锁定标签编辑
        noComment: false // 是否禁止评论
        noRate: false // 是否禁止评分
        rating: {blogId: 72850, total: 5, count: 1, average: 5}
        tags: {123: "标签名"} // 标签ID: 标签名称
        title: "资源标题"
        visitCount: 1234
        isFavorite: true // 是否收藏，仅登陆时返回
        topComments： [{
        author: "fff"
        commentId: 871611
        content: "<p>评论内容</p>"
        createDate: "2018-07-16T00:58:22.02"
        itemId: 72850
        rating: 0
        upvotes: 1
      }]
    }
    ```
 
* **Error Response:**

  * **Code:** 401 UNAUTHORIZED <br />
   需要登录才可访问的资源。


  * **Code:** 404 NOT FOUND <br />
   资源不存在或已被删除。

* **Sample Call:**

  `/api/blog/details/?id=72850`

* **Notes:**

  `rating`的类型为GmGard.Models.BlogRatingDisplay
  `topComments`为得分最高的5个评论（不包含评论回复）。类型详见评论列表API。

  ***投稿显示***

  投稿正文为用户生成的HTML。部分内容需要js与css支持。例如折叠内容，内嵌图片与图片放大等。
  内嵌图片的&lt;img&gt;HTML标签会附带data-index="0"的属性，数值对应imgeUrls数组的图片链接索引。

### 资源列表

* **/api/blog/list**

* **Method:** `GET`
  
*  **URL Params**

   `id=[non-negative integer]`分类ID

   `sort=Date|Date_desc|Visit|Visit_desc|Post|Post_desc|Rate|Rate_desc`按日期|访问数|最后投稿|评分

   `limit=[1-100]`每页个数

   `skip=[non-negative integer]`跳过个数，值应为limit的倍数

   `harmony=ture|false`若提供本参数，则只返回和谐/非和谐投稿（仅登陆有效）

* **Success Response:**
  
  GmGard.Models.App.Paged<GmGard.Models.App.BlogDetails>

  * **Code:** 200 <br />
    **Content:** 
    ```
    {
	  pageCount: 1000,
	  totalItemCount: 100000,
	  pageNumber: 1,
	  pageSize: 100,
	  skip: 0,
      items: [{
		author: {
			userId: 123
			userName: "用户名"
			nickName: "昵称"
			comment: "签名"
			points: 456  // 棒棒糖
			experience: 789  // 绅士度
			level: 3
			avatar: "头像url"
		},
		authorDesc: "用户签名"
		categoryId: 21  //分类ID
		parentCategoryId: 4 //父分类ID
		brief: "正文预览"
		content: "<p>正文</p>"
		createDate: "2017-10-26T01:49:14.16"
		id: 72850
		imageUrls: ["//static.gmgard.us/Images/upload/14525260149112764.jpg"]
		thumbUrl: ["//static.gmgard.us/Images/thumbs/14525260149112764.jpg"]
		links: [{name: "链接名称", url: "链接地址", pass: "提取码"}]
		isApproved: true // 是否通过审核，null为待审核
		lockTags: false // 是否锁定标签编辑
		noComment: false // 是否禁止评论
		noRate: false // 是否禁止评分
		rating: {blogId: 72850, total: 5, count: 1, average: 5}
		tags: {123: "标签名"} // 标签ID: 标签名称
		title: "资源标题"
		visitCount: 1234
		isFavorite: true // 是否收藏，仅登陆时返回
		topComments： [{
			author: "fff"
			commentId: 871611
			content: "<p>评论内容</p>"
			createDate: "2018-07-16T00:58:22.02"
			itemId: 72850
			rating: 0
			upvotes: 1
		}]
	  }, {
		// ...
	  }]
    }
    ```
 
* **Error Response:**

  * 无特殊错误，非登录状态也可访问。

* **Sample Call:**

  `/api/blog/list/?id=1`

* **Notes:**

  无id参数时，默认返回全部分类投稿。

  本API在登录的情况下等同于`/api/search/blogs?CategoryIds=id[&Harmony=harmony]`，非登录情况下等同于`/api/search/blogs?CategoryIds=id&Harmony=true`

  `topComments`为得分最高的5个评论（不包含评论回复）。类型详见评论列表API。

### 资源评分

* **/api/blog/rate**

* **Method:** `POST`
  
*  **Data Params**

   GmGard.Models.App.RateRequest
   ```
   {
     blogId: 123
     rating: 5
   }
   ```

* **Success Response:**
  
  GmGard.Models.App.RateResponse

  * **Code:** 200 <br />
    **Content:** 
    ```
    {
      status: "ok"
      message: "今天第一次评分。绅士度+1，棒棒糖+1"
      rating: {blogId: 72850, total: 5, count: 1, average: 5}
    }
    ```
 
* **Error Response:**

    **Code:** 400 <br/>
       ```
      {
	    error: "[status]"
      }
      ```

  * **status:** `"rated"` 用户已评分过。
  * **status:** `"login"` 需要登录才可以评分。
  * **status:** `"rated_today"` 本IP今日已评分过。（仅在开启游客每日IP评分时返回）
  * **status:** `"error"` 发生错误，无效的评分值，或对应的投稿可能已被删除。

* **Sample Call:**

  `$.post('/api/blog/rate', {blogId: 72850, rating: 5}, (r) => console.log(r));`

* **Notes:**

  **注意：** 请求中的`rating`为对应按钮位置（1: 评分为-1, 2: 评分为 0， 3: 评分为1，4: 评分为3，5: 评分为5。）

  返回值中`rating`的类型为`GmGard.Models.BlogRatingDisplay`
  
  另外此API目前并不检查投稿是否允许评分。

### 资源搜索

* **/api/blog/search**

* **Method:** `GET`|`POST`
  
* **Data Params**

  GmGard.Models.SearchModel
  
  ```
  {
   categoryIds: [1,2,3] // 想要查看的分类
   query: "综合搜索关键字"
   title: "标题关键字"
   titleMatchAny: false // 若为true，则搜索带有至少1个关键字（空格分开）的投稿，否则必须带有所有关键字。
   tags: "标签关键字"
   tagsMatchAny: false // 原理同titleMatchAny。
   tagIds: [1,2,3] // 仅查看含有这些标签的投稿。
   harmony: false // 若为true，仅返回可以游客访问的投稿。
   sort: "Date_desc" // 排序模式
   startDate: null // 投稿时间范围起始
   endDate: null // 投稿时间范围结束
   author: null // 仅查看该用户的投稿
  }
  ```
  
*  **URL Params**

   * `limit=10` 返回个数，1~100
   * `skip=0` 跳过个数，值应为limit的倍数，用于分页。

* **Success Response:**
  
  GmGard.Models.App.Paged<GmGard.Models.App.BlogPreview>

  * **Code:** 200 <br />
    **Content:** 
    同资源列表。
 
* **Error Response:**

  * 无特殊错误，非登录状态也可访问。

* **Notes:**

  支持的排序类型："Date"/"Date_desc"按日期，"Visit"/"Visit_desc"按浏览数，"Post"/"Post_desc"按评论数，"Rate"/"Rate_desc"按评分。
  综合搜索一并搜索投稿标题，标签与正文。当使用综合搜索时，同时还支持按相关度排序："Score"/"Score_desc"。

  此API与`api/search/blog`的参数相同，但返回内容不同。本API返回的格式与其他`api/blog`接口相同。

### 收藏列表

* **/api/blog/favorites**

* **Method:** `GET`
  
*  **URL Params**

   `id=[non-negative integer]`分类ID

   `sort=Date|Date_desc|Visit|Visit_desc|Post|Post_desc|Rate|Rate_desc|AddDate|AddDate_desc`按日期|访问数|最后投稿|评分|添加收藏日期

   `limit=[1-100]`每页个数

   `skip=[non-negative integer]`跳过个数，值应为limit的倍数

   `name=[string]`收藏列表用户名（默认当前用户）

* **Success Response:**
  
  GmGard.Models.App.Paged<GmGard.Models.App.BlogDetails>

  * **Code:** 200 <br />
    **Content:** 
    同资源列表。
 
* **Error Response:**

  * **Code:** 404 用户不存在。  
  * **Code:** 401 未登录。

* **Sample Call:**

  `/api/blog/favorites/?id=1`

* **Notes:**

  无id参数时，默认返回全部分类投稿。

  列表返回格式等同于资源列表。默认排序为`AddDate_desc`。

### 添加收藏

* **URL**

  /api/blog/favorite

* **Method:** `POST`

*  **URL Params**

   `id=[non-negative integer]`投稿ID

* **Success Response:** OK

* **Error Response:**

  **Code:** 404 投稿不存在。

### 删除收藏

* **URL**

  /api/blog/favorite

* **Method:** `DELETE`

*  **URL Params**

   `id=[non-negative integer]`投稿ID

* **Success Response:** OK

* **Error Response:**

  **Code:** 404 没有被收藏。

## 评论相关

### 评论列表

* **URL**

  /api/reply/comments

* **Method:** `GET`

*  **URL Params**

   `id=[non-negative integer]`投稿ID

   `commentType=Blog` 评论对象类型

   `sort=upvotes|date`按评论得分|评论日期降序排序（默认按日期排序）

   `limit=[1-100]`每页个数

   `skip=[non-negative integer]`跳过个数，值应为limit的倍数

* **Success Response:**

   GmGard.Models.App.Paged<GmGard.Models.App.Comment>
   ```
   {
	  pageCount: 10,
	  totalItemCount: 20,
	  pageNumber: 1,
	  pageSize: 10,
	  skip: 0,
	  items: [{
		author: "fff"
	    type: 'blog'
	    itemId: 123
		commentId: 456
	    content: '<p>评论内容</p>'
		createDate: "2018-07-16T00:58:22.02"
	    rating: 5
		upvotes: 2
		replies: [{
		  commentId: 456
		  replyId: 789
		  author: "eee"
		  content: '<p>回复内容</p>'
		  createDate: "2018-07-16T01:46:22"
		}]
	  }]
   }
   ```

* **Notes:**

  `replies`评论回复的类型为`GmGard.Models.App.CommentReply[]`。
  
  如果评论没有带评分，`rating`为`null`（不在响应中）。

### 添加评论

* **URL**

  /api/reply/comment

* **Method:** `POST`

*  **Data Params**

   GmGard.Models.App.AddCommentRequest
   ```
   {
     type: 'blog'
     itemId: 123
	 content: '<p>评论内容</p>'
	 rating: 5
   }
   ```

* **Success Response:**
  
  GmGard.Models.App.AddReplyResponse
  ```
  {
	commentId: 345
	message: "任务相关信息"
  }
  ```

* **Error Response:**
  
  **Code:** 400 <br/>
  ```
  {
	error: "您已经评过分了"
  }
  ```

* **Notes:**

  评论对象类型目前只支持'blog'。itemId即为对象（blog）的id。

  **注意：** 请求中的`rating`为对应按钮位置（1: 评分为-1, 2: 评分为 0， 3: 评分为1，4: 评分为3，5: 评分为5。）

  如果`rating`值无效，则忽略评分。

  返回值中`message`为任务相关信息（本日第一次评论/评分）。
  
  另外此API目前并不检查投稿是否允许评论或评分。

### 删除评论

* **URL**

  /api/reply/comment

* **Method:** `DELETE`

*  **URL Params**

   `id=[non-negative integer]`评论ID

* **Success Response:** OK

* **Error Response:**
  
  **Code:** 401 <br/>
  用户没有权限删除。

  **Code:** 404 <br/>
  评论不存在。

### 添加评论回复

* **URL**

  /api/reply/commentReply

* **Method:** `POST`

*  **Data Params**

   GmGard.Models.App.AddCommentReplyRequest
   ```
   {
     commentId: 123
	 content: '<p>评论内容</p>'
   }
   ```

* **Success Response:**
  
  GmGard.Models.App.AddReplyResponse
  ```
  {
	commentId: 123
	replyId: 456
	message: "任务相关信息"
  }
  ```

* **Error Response:**
  
  **Code:** 400 <br/>
  ```
  {
	error: "内容不能为空或纯图片"
  }
  ```

* **Notes:**

  返回值中`message`为任务相关信息（本日第一次评论）。

### 删除评论回复

* **URL**

  /api/reply/commentReply

* **Method:** `DELETE`

*  **URL Params**

   `id=[non-negative integer]`评论回复ID

* **Success Response:** OK

* **Error Response:**
  
  **Code:** 401 <br/>
  用户没有权限删除。

  **Code:** 404 <br/>
  评论不存在。

### 评论评价

* **URL**

  /api/reply/upvote

* **Method:** `POST`

*  **Data Params**

   ```
   {
     id: 123  // 评论id
	 value: -1|1  // 评价值
   }
   ```

* **Success Response:**
  
  ```
  {
	value: 10  // 当前评价
  }
  ```

* **Error Response:**
  
  **Code:** 400 <br/>
  如果评价值不为1或-1。

  **Code:** 404 <br/>
  评论不存在。

## 搜索相关

### 资源搜索/列表

* **URL**

  /api/search/blog

* **Method:** `GET` | `POST` 都行
  

* **Data Params**

  GmGard.Models.SearchModel
  
  ```
  {
   categoryIds: [1,2,3] // 想要查看的分类
   query: "综合搜索关键字"
   title: "标题关键字"
   titleMatchAny: false // 若为true，则搜索带有至少1个关键字（空格分开）的投稿，否则必须带有所有关键字。
   tags: "标签关键字"
   tagsMatchAny: false // 原理同titleMatchAny。
   tagIds: [1,2,3] // 仅查看含有这些标签的投稿。
   harmony: false // 若为true，仅返回可以游客访问的投稿。
   sort: "Date_desc" // 排序模式
   startDate: null // 投稿时间范围起始
   endDate: null // 投稿时间范围结束
   author: null // 仅查看该用户的投稿
  }
  ```
  
*  **URL Params**

   * `limit=10` 返回个数，1~100
   * `skip=0` 跳过个数，值应为limit的倍数，用于分页。

   若使用`GET`方法，需将参数以URL方式传递。注意数组类型需要以`?id=1&id=2&id=3`的形式发送。

* **Success Response:**
  
  GmGard.Models.App.Paged<GmGard.Models.App.BlogPreview>

  * **Code:** 200 <br />
    **Content:** 
    ```
    {
	  pageCount: 1000,
	  totalItemCount: 100000,
	  pageNumber: 1,
	  pageSize: 100,
	  skip: 0,
	  items: [{
		id: 123
		url: "http://gmgard.com/gm123"
		brief: "投稿第一行简介"
		thumbUrl: "缩略图url"
		title: "标题"
		createDate: "投稿日期"
		, {
		// ...
	  }]
	}
    ```
 
* **Error Response:**

  * **Code:** 401 UNAUTHORIZED <br />
   目前只允许登陆用户使用。

* **Sample Call:**

  `$.post('/search/blog?limit=10', { categoryIds: [1,2] }, (r) => console.log(r));`

* **Notes:**

  支持的排序类型："Date"/"Date_desc"按日期，"Visit"/"Visit_desc"按浏览数，"Post"/"Post_desc"按评论数，"Rate"/"Rate_desc"按评分。
  综合搜索一并搜索投稿标题，标签与正文。当使用综合搜索时，同时还支持按相关度排序："Score"/"Score_desc"。

## 分类相关

### 分类列表

* **URL**

  /api/category/list

* **Method:** `GET`
  
* **Success Response:**

  * **Code:** 200 <br />
    **Content:** 
    ```
    [{
      id: 1
      name: "分类名"
      parentId: null
    }, {
      id: 2
      name: "子分类"
      parentId: 1
    }]
    ```
* **Sample Call:**

  `/api/category/list`

* **Notes:**

  非登录状态下部分分类不可见。

### 分类新投稿数量

* **URL**

  /api/category/newItemCount

* **Method:** `GET`

*  **URL Params**

   * `since=2018-06-17T23:59:59` 计算新投稿数量的起始时间，默认为24小时，最久不得超过7天。
  
* **Success Response:**

  * **Code:** 200 <br />
    **Content:** 
    ```
	{
		since: "2018-06-17T23:59:59",  // 实际计算用的起始时间
		total: 123, // 新投稿总数
		byCategories: [{ // 按分类新投稿数
			id: 1,
			count: 12
		}, {
			id: 2,
			count: 23
		}]
	}
    ```
* **Sample Call:**

  `/api/category/newItemCount?since=2018-06-17T23:59:59`

* **Notes:**

  非登录状态下部分分类与投稿不参与统计。主分类投稿数量为子分类投稿数量的总和。
