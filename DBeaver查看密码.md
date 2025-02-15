- 进入 Preferences - General - Workspace, 找到 shwo full workspace path 后面的路径，用文件管理器打开路径
- 再进入目录下的 General - .dbeaver，将下面的credentials-config.json文件复制到linux环境
- 使用命令 openssl aes-128-cbc -d -K babb4a9f774ab853c96c2d653dfe544a -iv 00000000000000000000000000000000 -in credentials-c
onfig.json，可以输出明文明码