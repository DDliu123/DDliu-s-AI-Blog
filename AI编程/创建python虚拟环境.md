查看python版本号
python --version
1. 创建项目文件夹（示例：my_python_project）
mkdir my_python_project
2. 进入该文件夹
cd my_python_project
3. Windows/Mac/Linux通用（注意：Windows用python，Mac/Linux可能需要python3）
python -m venv venv  # 第一个venv是模块名，第二个是虚拟环境文件夹名
4.激活虚拟环境
venv\Scripts\Activate.ps1（需先执行：Set-ExecutionPolicy RemoteSigned -Scope CurrentUser）
5.安装LangChain
pip install -U langchain
4. 安装LangChain OpenAI
pip install -U langchain-openai
5. 安装环境变量管理dotenv
pip install dotenv
DEEPSEEK_API_KEY="sk-1fd1f70b01844e949c604da912fb35dd"
DEEPSEEK_BASE_URL="https://api.deepseek.com/v1"
pip install 'langchain[openai]' dotenv

