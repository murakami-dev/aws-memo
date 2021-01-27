# やること
- https://docs.aws.amazon.com/ja_jp/pinpoint/latest/developerguide/tutorials-importing-data.html

# 手順できになったこと
- ステップ1のS3バケット、暗号化できないのか？


# はまったこと
- ステップ3のスクリプト実行で`このシステムではスクリプトの実行が無効になっているため、ファイル C:\Users\****を読み込むことができません。`
  - [PowerShellでvirtualenvを使うには](https://qiita.com/ryu22e/items/520b35db6a444d8289da)
  - >これは、PowerShellのスクリプト実行ポリシーが原因です。現在の実行ポリシーは`Get-ExecutionPolicy`コマンドで確認できます。
```
PS C:\Users\user\pinpoint-import> Set-ExecutionPolicy RemoteSigned -Scope Process
PS C:\Users\user\pinpoint-import> .\venv\Scripts\activate
(venv) PS C:\Users\user\pinpoint-import>
```

### 参考ステップ3
```
PS C:\Users\user\pinpoint-import> .\venv\Scripts\activate
.\venv\Scripts\activate : このシステムではスクリプトの実行が無効になっているため、ファイル C:\Users\user\pinpoint-impor
t\venv\Scripts\Activate.ps1 を読み込むことができません。詳細については、「about_Execution_Policies」(https://go.microso
ft.com/fwlink/?LinkID=135170) を参照してください。
発生場所 行:1 文字:1
+ .\venv\Scripts\activate
+ ~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : セキュリティ エラー: (: ) []、PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess
PS C:\Users\user\pinpoint-import> Set-ExecutionPolicy RemoteSigned -Scope Process
PS C:\Users\user\pinpoint-import> .\venv\Scripts\activate
(venv) PS C:\Users\user\pinpoint-import> pip install s3fs jmespath s3transfer six python-dateutil docutils
Collecting docutils
  Downloading docutils-0.16-py2.py3-none-any.whl (548 kB)
     |████████████████████████████████| 548 kB 1.3 MB/s
Collecting jmespath
  Using cached jmespath-0.10.0-py2.py3-none-any.whl (24 kB)
Collecting python-dateutil
  Using cached python_dateutil-2.8.1-py2.py3-none-any.whl (227 kB)
Collecting six
  Using cached six-1.15.0-py2.py3-none-any.whl (10 kB)
Collecting s3fs
  Downloading s3fs-0.5.2-py3-none-any.whl (22 kB)
Collecting aiobotocore>=1.0.1
  Downloading aiobotocore-1.2.0.tar.gz (47 kB)
     |████████████████████████████████| 47 kB 3.2 MB/s
Collecting aiohttp>=3.3.1
  Downloading aiohttp-3.7.3-cp38-cp38-win_amd64.whl (634 kB)
     |████████████████████████████████| 634 kB 6.8 MB/s
Collecting aioitertools>=0.5.1
  Downloading aioitertools-0.7.1-py3-none-any.whl (20 kB)
Collecting async-timeout<4.0,>=3.0
  Downloading async_timeout-3.0.1-py3-none-any.whl (8.2 kB)
Collecting attrs>=17.3.0
  Downloading attrs-20.3.0-py2.py3-none-any.whl (49 kB)
     |████████████████████████████████| 49 kB 524 kB/s
Collecting botocore<1.19.53,>=1.19.52
  Downloading botocore-1.19.52-py2.py3-none-any.whl (7.2 MB)
     |████████████████████████████████| 7.2 MB 504 kB/s
Collecting chardet<4.0,>=2.0
  Using cached chardet-3.0.4-py2.py3-none-any.whl (133 kB)
Collecting fsspec>=0.8.0                                                                                                  Downloading fsspec-0.8.5-py3-none-any.whl (98 kB)                                                                          |████████████████████████████████| 98 kB 3.2 MB/s                                                                  Collecting multidict<7.0,>=4.5                                                                                            Downloading multidict-5.1.0-cp38-cp38-win_amd64.whl (48 kB)                                                                |████████████████████████████████| 48 kB 2.7 MB/s
Collecting typing-extensions>=3.6.5
  Downloading typing_extensions-3.7.4.3-py3-none-any.whl (22 kB)
Collecting urllib3<1.27,>=1.25.4
  Downloading urllib3-1.26.3-py2.py3-none-any.whl (137 kB)
     |████████████████████████████████| 137 kB 656 kB/s
Collecting wrapt>=1.10.10
  Using cached wrapt-1.12.1.tar.gz (27 kB)
Collecting yarl<2.0,>=1.0
  Downloading yarl-1.6.3-cp38-cp38-win_amd64.whl (125 kB)
     |████████████████████████████████| 125 kB 3.3 MB/s
Collecting idna>=2.0
  Downloading idna-3.1-py3-none-any.whl (58 kB)
     |████████████████████████████████| 58 kB 1.7 MB/s
Collecting s3transfer
  Downloading s3transfer-0.3.4-py2.py3-none-any.whl (69 kB)
     |████████████████████████████████| 69 kB 4.5 MB/s
Building wheels for collected packages: aiobotocore, wrapt
  Building wheel for aiobotocore (setup.py) ... done
  Created wheel for aiobotocore: filename=aiobotocore-1.2.0-py3-none-any.whl size=45550 sha256=a8d4aa67e792d1f7677563f8d173d769e28a756f3d56a293c61ae46b9dbde11c
  Stored in directory: c:\users\user\appdata\local\packages\pythonsoftwarefoundation.python.3.8_qbz5n2kfra8p0\localcache\local\pip\cache\wheels\03\72\9a\4212a33194f60ef51f0b3659f153d5c3284e3e8b67481105fb
  Building wheel for wrapt (setup.py) ... done
  Created wheel for wrapt: filename=wrapt-1.12.1-py3-none-any.whl size=19553 sha256=3edc8cfe7ebf7a26633d143fdfeb810be79c45963bd9d73d0fdc7f1622088249
  Stored in directory: c:\users\user\appdata\local\packages\pythonsoftwarefoundation.python.3.8_qbz5n2kfra8p0\localcache\local\pip\cache\wheels\5f\fd\9e\b6cf5890494cb8ef0b5eaff72e5d55a70fb56316007d6dfe73
Successfully built aiobotocore wrapt
Installing collected packages: six, multidict, idna, yarl, urllib3, typing-extensions, python-dateutil, jmespath, chardet, attrs, async-timeout, wrapt, botocore, aioitertools, aiohttp, fsspec, aiobotocore, s3transfer, s3fs, docutils
Successfully installed aiobotocore-1.2.0 aiohttp-3.7.3 aioitertools-0.7.1 async-timeout-3.0.1 attrs-20.3.0 botocore-1.19.52 chardet-3.0.4 docutils-0.16 fsspec-0.8.5 idna-3.1 jmespath-0.10.0 multidict-5.1.0 python-dateutil-2.8.1 s3fs-0.5.2 s3transfer-0.3.4 six-1.15.0 typing-extensions-3.7.4.3 urllib3-1.26.3 wrapt-1.12.1 yarl-1.6.3
WARNING: You are using pip version 20.3.3; however, version 21.0 is available.
You should consider upgrading via the 'C:\Users\user\pinpoint-import\venv\Scripts\python.exe -m pip install --upgrade pip' command.
(venv) PS C:\Users\user\pinpoint-import> deactivate
PS C:\Users\user\pinpoint-import> cd .\venv\Lib\site-packages\
>> Compress-Archive -Path dateutil, docutils, jmespath, s3fs, s3transfer, six.py `
>> -DestinationPath ..\..\..\pinpoint-importer.zip ; cd ..\..\..
>>
PS C:\Users\user\pinpoint-import>
```
