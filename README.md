## aws-ssh-loader

AWS EC2 정보를 로컬의 ssh-config로 sync 해주는 cli command tool 입니다.
VSCode 에서 ssh 에 접속할 때 편하게 사용할 수 있습니다.
현재 켜져있는 인스턴스의 리스트들만 받아옵니다.

### How to Use

먼저 로컬의 awscli 가 셋업되어 있어야 합니다.

``` shell
cd ~/
git clone git@github.com:dalphakr/aws-ssh-loader.git

# 환경 변수 등록, ~/.zshrc 등에 다음을 등록합니다.
$ export PATH=$PATH:~/aws-ssh-loader

# 일부 key pair 의 경우 jq 라는 parser 에서 실제 identity file 위치로 변경해줍니다.
# ec2sync 에 현재 로컬에 갖고 있는 pem 파일 위치를 명시합니다.
# in ec2sync file
export TEST_INSTANCE_KEY_PATH=<insert your keypair path>
```

VSCode 에서 cmd + shift + P 로 preference 를 열고 `Remote SSH: Open SSH Configuration Files...` 에서 설정으로 들어가
Config file 의 path 를 `~/aws-ssh-loader/ssh-config` 로 등록합니다.

이후 shell 에서 `ec2sync` 를 치면 ssh config 가 업데이트 됩니다.


#### SubModule: json-to-ssh_config
==================

[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/ambv/black)

A simple JSON to ssh_config file generator. The basic usage model is that you
stick all the JSON files into a certain folder (the default value is
`~/.ssh/confs/`, overrideable via the `-s` parameter), which then in turn get
loaded/validated via Python's `json.load()`, and translated to
[ssh_config(5)](http://manpg.es/ssh_config) file format.

There are some assumptions that the script makes:
* All JSON files are in the same folder
* All JSON files end with a `.conf` suffix
* All JSON files get {displayed,written} alphabetically, except for `global.conf`,
which is always last

There are two basic syntax models that are supported:

```json
{
    "Hosts":
    [{
        "Host": "aliasname",
        "HostName": "fqdnname",
        "Port": 12345
    }]
}
```
Which results in:
```sh
$ ./gensshconf.py -s .
# Content from ./demo1.conf
Host aliasname
  HostName fqdnname
  Port 12345

```
And:
```json
{
    "Options":
    {
        "IdentityFile": "pathtoidentityfile",
        "Port": 12345
    },
    "Hosts":
    [{
        "Host": "aliasname1",
        "HostName": "fqdnname1"
    },
    {
        "Host": "aliasname2",
        "HostName": "fqdnname2",
        "Port": 23456
    }]
}
```
Which results in:
```sh
$ ./gensshconf.py -s .
# Content from ./demo2.conf
Host aliasname1
  HostName fqdnname1
  IdentityFile pathtoidentityfile
  Port 12345
Host aliasname2
  HostName fqdnname2
  Port 23456
  IdentityFile pathtoidentityfile

```
The `global.conf` file is a special case, its intended usage is to contain the
`Host *` settings that apply to all hosts:
```json
{
    "Hosts":
    [{
        "Host": "*",
        "ControlMaster": "auto",
        "ControlPath": "somepath",
        "ForwardAgent": "no",
        "IdentityFile": "somefile",
        "IdentitiesOnly": "yes",
        "Protocol": 2,
        "User": "username",
        "VisualHostKey": "yes"
    }]
}
```
Which results in:
```sh
$ ./gensshconf.py -s .
# Content from ./global.conf
Host *
  ControlMaster auto
  ControlPath somepath
  ForwardAgent no
  IdentityFile somefile
  IdentitiesOnly yes
  Protocol 2
  User username
  VisualHostKey yes

```
Note that there are some aspects of ssh_config(5) which are not supported
directly, such as overriding a set of hosts via wildcards (`*.example.com`).
That could be hacked around via a combination of some smart file naming and
existing syntax.

There are two supported output modes, `screen` and `file`, controlled via the
`-o` parameter. The default value is `screen`. When used with the `-o file`
parameter, the default output file is `outfile-example` in the script
directory. This can be overridden via the `-f` parameter:
```sh
$ ./gensshconf.py -o file -f ~/.ssh/config
```
This is still a fairly rudimentary implementation, but it seems to work
properly for me. YMMV disclaimer is implied, as always. :)
