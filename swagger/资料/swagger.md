# define a path with parameter reference
/path:
   get:
      parameters:
         - $ref: "#/parameters/limitParam"

# define reusable parameters:
parameters:
   limitParam:
      name: limit
      in: query
      description: Limits the number of returned results
      required: false
      type: number
      format: int32


使用$ ref
在记录API时，通常会使用跨若干API资源使用的某些功能。在这种情况下，您可以为这些元素创建一个片段，以便在需要时多次使用它们。

借助OpenAPI 3.0，您可以引用托管在任何位置的定义。它可以是同一台服务器，也可以是另一台服务器 - 例如GitHub，SwaggerHub等等。

要引用定义，请使用$ref关键字：

$ref: 'reference to definition'
例如，假设您有以下模式对象，您想在响应中使用它：

JSON示例   YAML示例
"components": {
  "schemas": {
    "user": {
      "properties": {
        "id": {
          "type": "integer"
        },
        "name": {
          "type": "string"
        }
      }
    }
  }
}
components:
  schemas:
    User:
      properties:
        id:
          type: integer
        name:
          type: string
要引用该对象，您需要添加$ref相应的路径到您的响应中：

JSON示例   YAML示例
"responses": {
  "200": {
    "description": "The response",
    "schema": {
      "$ref": "#/components/schemas/user" 
    }
  }
}
responses:
  '200':
    description: The response
    schema: 
      $ref: '#/components/schemas/User'
该值$ref使用JSON引用表示法，而以此开头的部分#使用JSON指针表示法。这个表示法可以让你指定目标文件或你想引用的文件的特定部分。在前面的例子，#/components/schemas/User指的是从当前文档的根的分辨开始，然后找到的值components，schemas和User一个接一个的。

$ ref语法
根据RFC3986，$ref字符串值（JSON引用）应该包含一个URI，它标识了您引用的JSON值的位置。如果字符串值不符合URI语法规则，则会在解析过程中导致错误。除$refJSON引用对象以外的任何成员都将被忽略。

在特定情况下，请查看此列表以查看JSON参考的示例值：

本地参考 - $ref: '#/definitions/myElement'
#指转到当前文档的根，然后找到元素definitions和myElement一个一个之后。
远程参考 - $ref: 'document.json'
使用位于同一服务器上且位于同一位置的整个文档。
位于同一台服务器上的文档元素 -$ref: 'document.json#/myElement'
位于父文件夹中的文档元素 -$ref: '../document.json#/myElement'
位于另一个文件夹中的文档的元素 -$ref: '../another-folder/document.json#/myElement'
URL参考 - $ref: 'http://path/to/your/resource'
使用位于不同服务器上的整个文档。
存储在不同服务器上的文档的特定元素 -$ref: 'http://path/to/your/resource.json#myElement'
不同服务器上的文档使用相同的协议（例如HTTP或HTTPS） -$ref: '//anotherserver.com/files/example.json'
注意：使用本地引用（如#/components/schemas/UserYAML）时，请将该值放在引号中：'#/components/schemas/User'。否则，它将被视为评论。

转义字符
/并且~是JSON指针中的特殊字符，并且在逐字地使用时需要转义（例如，在路径名中）。

字符 逃脱
〜  〜0
/  〜1
例如，要引用路径/blogs/{blog_id}/new~posts，您可以使用：

$ref: '#/paths/~1blogs~1{blog_id}~1new~0posts'
注意事项
可以使用$ ref的地方
一个常见的误解是，$ref它在OpenAPI规范文件的任何位置都是允许的。实际上$ref只允许在OpenAPI 3.0规范中明确指出该值可能是引用的地方。

例如，$ref不能在本info节中直接使用paths：

openapi: 3.0.0

# Incorrect!
info:
  $ref: info.yaml
paths:
  $ref: paths.yaml
但是，您可以$ref单独使用路径，如下所示：

paths:
  /users:
    $ref: '../resources/users.yaml'
  /users/{userId}:
    $ref: '../resources/users-by-id.yaml'
$ ref和兄弟元素
a的任何兄弟元素$ref都被忽略。这是因为$ref通过取代它自己和所有与它指向的定义相关的作品。

考虑这个例子：

components:
  schemas:
    Date:
      type: string
      format: date

    DateWithExample:
      $ref: '#/components/schemas/Date'
      description: Date schema extended with a `default` value... Or not?
      default: 2000-01-01
在第二个模式中，description和default属性被忽略，所以这个模式结束与引用的Date模式完全相同。

