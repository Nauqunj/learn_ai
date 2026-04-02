

报错：fatal: unable to access 'https://github.com/Nauqunj/learn_ai.git/': Failed to connect to github.com port 443 after 21097 ms: Timed out

问题分析
Git 所设端口与系统代理不一致，需重新设置

# 注意修改成自己的IP和端口号
git config --global http.proxy http://127.0.0.1:7890 
git config --global https.proxy http://127.0.0.1:7890