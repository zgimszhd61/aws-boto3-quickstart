# aws-boto3-quickstart
如果你想使用 Python 的 `boto3` 库来存储文件到 AWS S3，这里有一个简单的例子来帮助你快速开始。这个例子包括安装 `boto3`，配置 AWS 访问凭证，以及上传文件到 S3 的步骤。

### 第一步：安装 `boto3`

首先，你需要确保已经安装了 `boto3`。可以通过 pip 安装：

```bash
pip install boto3
```

### 第二步：配置 AWS 凭证

在使用 `boto3` 与 AWS 服务进行交互之前，你需要配置你的 AWS 访问凭证。你可以通过多种方式配置凭证，最简单的一种是在你的本地机器上设置 AWS 访问密钥。你可以在 `~/.aws/credentials` 文件中添加以下内容（如果没有这个文件，你需要创建它）：

```
[default]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_SECRET_KEY
```

确保替换 `YOUR_ACCESS_KEY` 和 `YOUR_SECRET_KEY` 为你的 AWS 访问密钥和秘密访问密钥。

### 第三步：上传文件到 S3

以下是一个 Python 脚本，它使用 `boto3` 来上传一个文件到指定的 S3 桶：

```python
import boto3

# 创建 S3 客户端
s3 = boto3.client('s3')

# 指定桶名称
bucket_name = 'your-bucket-name'

# 指定要上传的文件名和在 S3 上的存储路径
file_name = 'local-file.txt'
s3_file_name = 'uploaded-file.txt'

# 上传文件
s3.upload_file(file_name, bucket_name, s3_file_name)

print(f'File {file_name} has been uploaded to {bucket_name}/{s3_file_name}')
```

替换 `your-bucket-name`, `local-file.txt` 和 `uploaded-file.txt` 为你的 S3 桶名称、本地文件名以及你想要保存在 S3 上的文件名。

这个脚本将会连接到 AWS，然后上传指定的文件到你的 S3 桶中。确保你的 AWS 账户有足够的权限来执行这些操作。如果你遇到任何权限问题，请检查你的 IAM 策略设置。

# 如何生成临时下载链接

要使用`boto3`库从AWS S3获取数据并生成临时下载链接，你可以遵循以下步骤。这个例子将展示如何连接到S3，获取一个对象，并为这个对象创建一个有效期为60分钟的临时下载链接。

### 安装 Boto3

首先，你需要确保已经安装了`boto3`库。如果尚未安装，可以通过以下命令安装：

```bash
pip install boto3
```

### 配置 AWS 凭证

你需要配置AWS的访问密钥和密钥ID，这可以通过多种方式完成，例如通过环境变量、配置文件或在代码中直接指定。

#### 方法1：使用环境变量

```bash
export AWS_ACCESS_KEY_ID="your_access_key_id"
export AWS_SECRET_ACCESS_KEY="your_secret_access_key"
export AWS_SESSION_TOKEN="your_session_token"  # 可选，只有在使用临时凭证时需要
```

#### 方法2：使用配置文件

将以下内容添加到`~/.aws/credentials`文件：

```
[default]
aws_access_key_id = your_access_key_id
aws_secret_access_key = your_secret_access_key
```

#### 方法3：在代码中直接指定

在代码中直接设置凭证（较不推荐，因为可能导致凭证泄露）。

### 示例代码

以下是一个Python脚本示例，展示如何从S3桶中获取一个对象并创建一个临时下载链接：

```python
import boto3
from botocore.exceptions import NoCredentialsError

def create_presigned_url(bucket_name, object_name, expiration=3600):
    """生成一个预签名URL以允许下载S3对象。
    
    :param bucket_name: S3桶的名称
    :param object_name: S3对象的键名
    :param expiration: 链接的有效期，以秒为单位
    :return: 预签名URL或错误信息
    """
    # 创建S3客户端
    s3_client = boto3.client('s3')
    try:
        response = s3_client.generate_presigned_url('get_object',
                                                    Params={'Bucket': bucket_name,
                                                            'Key': object_name},
                                                    ExpiresIn=expiration)
    except NoCredentialsError:
        return "Error: No credentials provided"
    except Exception as e:
        return str(e)

    return response

# 使用示例
bucket_name = 'your_bucket_name'
object_key = 'your_object_key'
url = create_presigned_url(bucket_name, object_key, 3600)
print("Generated Presigned URL: ", url)
```

这段代码定义了一个函数`create_presigned_url`，该函数接收S3桶名称、对象键名和链接的有效时间（默认为1小时），并返回一个临时链接，你可以通过这个链接在不公开你的AWS凭证的情况下访问S3对象。

请确保将`bucket_name`和`object_key`替换为你的具体值。你可以调整`expiration`参数以设置不同的链接有效时间。
