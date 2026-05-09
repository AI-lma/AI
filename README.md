法律文书智能起草助手
基于 DeepSeek API + 企业微信的法律文书 Agent，支持法律意见书、法律备忘录、诉讼方案三类文书的对话式生成。

环境要求
- Python 3.9+
- Windows 系统
- 企业微信账号（需要管理员权限）

安装依赖
pip install flask wechatpy openai python-docx requests pycryptodome


配置
1. 企业微信后台创建应用
- 登录 work.weixin.qq.com
- 应用管理 → 自建 → 创建应用
- 记录 AgentId、Secret、企业ID
2. 修改 wecom.py
'''python
CORP_ID = "你的企业ID"
CORP_SECRET = "你的Secret"
AGENT_ID = 你的AgentId
'''
3. 修改 agent.py
'''python
DEEPSEEK_API_KEY = "你的DeepSeek API Key"
'''
4. 修改 app.py
'''python
CORP_ID = "你的企业ID"
TOKEN = "你设置的Token"
ENCODING_AES_KEY = "你的EncodingAESKey"
'''

运行
第一步：启动Flask服务**
'''bash
python app.py
'''
第二步：启动Cloudflare Tunnel（新开一个命令行窗口）**
'''bash
cloudflared-windows-amd64.exe tunnel --url http://localhost:5000
'''
记录生成的公网地址，例如：
'''
https://xxx-xxx.trycloudflare.com
'''
第三步：配置企业微信接收消息**
- 应用管理 → 法律文书助手 → 接收消息 → 设置API接收
- URL填写：https://xxx-xxx.trycloudflare.com/webhook
- Token：与app.py中一致
- EncodingAESKey：与app.py中一致
- 保存
第四步：配置可信IP
- 运行以下命令获取当前出口IP：
'''bash
python -c "import requests; from wecom import get_access_token; token=get_access_token(); r=requests.post(f'https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token={token}', json={'touser':'你的企业微信ID','msgtype':'text','agentid':你的AgentId,'text':{'content':'test'}}); print(r.json())"
'''
- 将返回的 from ip 添加到：我的企业 → 企业信息 → 企业可信IP

使用
在企业微信中找到「法律文书助手」，直接发送消息即可：
- 发送「帮我起草一份法律意见书」
- 发送「帮我写一份法律备忘录」
- 发送「帮我制定诉讼方案」
生成的Word文档保存在项目目录的 `outputs/` 文件夹中。

项目结构
'''
Agent/
├── app.py          # Flask服务器，接收企业微信消息
├── agent.py        # Agent对话逻辑，调用DeepSeek API
├── wecom.py        # 企业微信消息收发
├── docx_gen.py     # 生成Word文档
└── outputs/        # 生成的文书保存目录
'''
