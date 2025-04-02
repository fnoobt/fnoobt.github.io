---
title:  FRR 北向 gRPC
author: fnoobt
date: 2025-04-02 18:17:00 +0800
categories: [Network,路由协议]
tags: [network,frr,grpc]
---

## 配置北向gRPC
开启FRR中[北向gRPC](https://docs.frrouting.org/projects/dev-guide/en/latest/grpc.html)有两个条件：
1. 运行`configure`时指定`--enable-grpc`参数
2. 在启动每个守护进程时，在对应的`daemon`参数中指定`-M grpc:<port>` 加载 gRPC 模块以及绑定到哪个端口

编辑`/etc/frr/daemons`，在对应的daemon后面添加`-M grpc:<port>`参数，如port不填则默认为50051。
```yaml
bgpd_options="  -A 127.0.0.1 -M grpc"
```
{: file="/etc/frr/daemons" }

配置完毕后可使用netstat命令验证gRPC服务端端口是否被监听
```bash
sudo netstat -anlp|grep 50051
```

frr官网提供了 [C++](https://docs.frrouting.org/projects/dev-guide/en/latest/grpc.html#c-example)、[python](https://docs.frrouting.org/projects/dev-guide/en/latest/grpc.html#python-example)、[Ruby](https://docs.frrouting.org/projects/dev-guide/en/latest/grpc.html#ruby-example)这3种语言的示例，其他语言的支持请参考[gRPC官方](https://grpc.io/)。

## C++示例
### 生成 FRR 北向绑定
```bash
# Install gRPC (e.g., on Ubuntu 20.04)
sudo apt-get install libgrpc++-dev libgrpc-dev

mkdir /tmp/frr-cpp
cd grpc

protoc --cpp_out=/tmp/frr-cpp \
       --grpc_out=/tmp/frr-cpp \
       -I $(pwd) \
       --plugin=protoc-gen-grpc=`which grpc_cpp_plugin` \
        frr-northbound.proto
```

执行后会在`/tmp/frr-cpp`中生成`frr-northbound.grpc.pb.cc`、`frr-northbound.grpc.pb.h`、`frr-northbound.pb.cc`、`frr-northbound.pb.h`四个C++文件。

### 获取版本和接口
下面的示例脚本，用于打印所有发现的接口。

```cpp
#include <string>
#include <sstream>
#include <grpc/grpc.h>
#include <grpcpp/create_channel.h>
#include "frr-northbound.pb.h"
#include "frr-northbound.grpc.pb.h"

int main() {
    frr::GetRequest request;
    frr::GetResponse reply;
    grpc::ClientContext context;
    grpc::Status status;

    auto channel = grpc::CreateChannel("localhost:50051",
                                     grpc::InsecureChannelCredentials());
    auto stub = frr::Northbound::NewStub(channel);

    request.set_type(frr::GetRequest::ALL);
    request.set_encoding(frr::JSON);
    request.set_with_defaults(true);
    request.add_path("/frr-interface:lib");
    auto stream = stub->Get(&context, request);

    std::ostringstream ss;
    while (stream->Read(&reply))
      ss << reply.data().data() << std::endl;

    status = stream->Finish();
    assert(status.ok());
    std::cout << "Interface Info:\n" << ss.str() << std::endl;
}
```
{: file="test.cpp" }

编译和运行该程序
```bash
# 编译C++测试程序
g++ -o test test.cpp frr-northbound.grpc.pb.cc frr-northbound.pb.cc -lgrpc++ -lprotobuf

# 运行测试程序
./test
```

示例输出：
```json
Interface Info:
{
  "frr-interface:lib": {
    "interface": [
      {
        "name": "lo",
        "vrf": "default",
        "state": {
          "if-index": 1,
          "mtu": 0,
          "mtu6": 65536,
          "speed": 0,
          "metric": 0,
          "phy-address": "00:00:00:00:00:00"
        },
        "frr-zebra:zebra": {
          "state": {
            "up-count": 0,
            "down-count": 0,
            "ptm-status": "disabled"
          }
        }
      },
      {
        "name": "r1-eth0",
        "vrf": "default",
        "state": {
          "if-index": 2,
          "mtu": 1500,
          "mtu6": 1500,
          "speed": 10000,
          "metric": 0,
          "phy-address": "02:37:ac:63:59:b9"
        },
        "frr-zebra:zebra": {
          "state": {
            "up-count": 0,
            "down-count": 0,
            "ptm-status": "disabled"
          }
        }
      }
    ]
  },
  "frr-zebra:zebra": {
    "workqueue-hold-timer": 10,
    "zapi-packets": 1000,
    "import-kernel-table": {
      "distance": 15
    },
    "dplane-queue-limit": 200
  }
}
```

## Python示例
### 生成 FRR 北向绑定

1. 安装 python3 虚拟环境功能，例如
```bash
sudo apt-get install python3-venv
```

2. 为 python grpc 创建虚拟环境并激活
```bash
python3 -m venv venv-grpc
source venv-grpc/bin/activate
```

3. 安装python的grpc依赖
```bash
pip3 install grpcio grpcio-tools
```

4. 使用`frr/grpc`目录中的`frr-northbound.proto`生成python代码
```bash
mkdir /tmp/frr-python
cd grpc

python3 -m grpc_tools.protoc  \
        --python_out=/tmp/frr-python \
        --grpc_python_out=/tmp/frr-python \
        -I $(pwd) \
        frr-northbound.proto
```

执行后会在`/tmp/frr-python`中生成`frr_northbound_pb2_grpc.py`和`frr_northbound_pb2.py`两个python文件。

> Python使用虚拟环境有利于解决项目间依赖冲突并实现环境隔离，为每个项目创建独立环境，可以使用不同的库和Python版本。切换环境`workon env_name`，退出环境`deactivate`。
{: .prompt-info } 

### 获取功能和接口
下面的示例脚本，用于打印 Python 发现的功能和所有接口。这演示了从 gRPC 获得的 2 种不同的 RPC 结果，即接口状态的 Unary (GetCapabilities) 和 Streaming (Get)。
```py
import grpc
import frr_northbound_pb2
import frr_northbound_pb2_grpc

# 此处的ip填入frrouting服务的具体ip
channel = grpc.insecure_channel('localhost:50051')
stub = frr_northbound_pb2_grpc.NorthboundStub(channel)

# Print Capabilities
request = frr_northbound_pb2.GetCapabilitiesRequest()
response = stub.GetCapabilities(request)
print(response)

# Print Interface State and Config
request = frr_northbound_pb2.GetRequest()
request.path.append("/frr-interface:lib")
request.type=frr_northbound_pb2.GetRequest.ALL
request.encoding=frr_northbound_pb2.XML

for r in stub.Get(request):
    print(r.data.data)
```

示例输出：
```xml
frr_version: "7.7-dev-my-manual-build"
rollback_support: true
supported_modules {
  name: "frr-filter"
  organization: "FRRouting"
  revision: "2019-07-04"
}
supported_modules {
  name: "frr-interface"
  organization: "FRRouting"
  revision: "2020-02-05"
}
[...]
supported_encodings: JSON
supported_encodings: XML

<lib xmlns="http://frrouting.org/yang/interface">
  <interface>
    <name>lo</name>
    <vrf>default</vrf>
    <state>
      <if-index>1</if-index>
      <mtu>0</mtu>
      <mtu6>65536</mtu6>
      <speed>0</speed>
      <metric>0</metric>
      <phy-address>00:00:00:00:00:00</phy-address>
    </state>
    <zebra xmlns="http://frrouting.org/yang/zebra">
      <state>
        <up-count>0</up-count>
        <down-count>0</down-count>
      </state>
    </zebra>
  </interface>
  <interface>
    <name>r1-eth0</name>
    <vrf>default</vrf>
    <state>
      <if-index>2</if-index>
      <mtu>1500</mtu>
      <mtu6>1500</mtu6>
      <speed>10000</speed>
      <metric>0</metric>
      <phy-address>f2:62:2e:f3:4c:e4</phy-address>
    </state>
    <zebra xmlns="http://frrouting.org/yang/zebra">
      <state>
        <up-count>0</up-count>
        <down-count>0</down-count>
      </state>
    </zebra>
  </interface>
</lib>
```

## FRR 北向绑定 (gRPC) 支持的功能与接口
FRR 的北向接口通过 gRPC 提供了网络设备的配置管理、状态监控和事务操作能力，主要支持以下功能：
- **​配置管理**：候选配置的创建、编辑、加载和提交
- **​数据查询**：配置数据、状态数据或两者的获取
- **​事务控制**：两阶段提交（2PC）的事务管理
- **​锁机制**：配置的并发访问控制
- **​元数据查询**：FRR 版本、支持模块和编码格式的能力查询
- **​YANG RPC 执行**：自定义操作的执行

### 详细接口与功能如下:
- **GetCapabilities**
  + 功能：获取FRR支持的能力信息，包括版本、支持的YANG模块、编码格式（JSON/XML）和回滚功能。
  + 请求/响应：`GetCapabilitiesRequest`/`GetCapabilitiesResponse`
  + 响应字段：
    * `frr_version`：FRR版本号
    * `rollback_support`：是否支持配置回滚
    * `supported_modules`：支持的YANG模块列表（含名称、组织、修订版本）
    * `supported_encodings`：支持的数据编码格式（JSON或XML）
- **Get**
  + 功能：从FRR中获取配置数据（`CONFIG`）、状态数据（`STATE`）或全部数据（`ALL`），支持流式传输。
  + 请求/响应：`GetRequest`/`GetResponse`
  + 关键参数：
    * `type`：数据类型（`ALL`/`CONFIG`/`STATE`）
    * `path`：YANG路径（用于过滤数据）
    * `encoding`：数据编码格式（JSON或XML）
  + 响应字段：
    * `data`：返回的配置或状态数据（封装在`DataTree`中）
    * `timestamp`：数据时间戳
- **CreateCandidate**
  + 功能：创建一个候选配置（基于当前运行配置的副本），返回候选配置ID。
  + 请求/响应：`CreateCandidateRequest`/`CreateCandidateResponse`
  + 响应字段：`candidate_id`（候选配置的唯一标识）
- **DeleteCandidate**
  + 功能：删除指定的候选配置。
  + 请求/响应：`DeleteCandidateRequest`/`DeleteCandidateResponse`
  + 关键参数：`candidate_id`
- **UpdateCandidate**
  + 功能：将候选配置基于最新的运行配置进行更新（自动解决冲突）。
  + 请求/响应：`UpdateCandidateRequest`/`UpdateCandidateResponse`
  + 响应字段：`candidate_id`
- **EditCandidate**
  + 功能：编辑候选配置，支持批量添加/删除配置项。
  + 请求/响应：`EditCandidateRequest`/`EditCandidateResponse`
  + 关键参数：
    * `candidate_id`
    * `update`：需更新的路径-值对列表（`PathValue`）
    * `delete`：需删除的路径列表（`PathValue`）
- **LoadToCandidate**
  + 功能：将完整配置加载到候选配置中，支持合并（`MERGE`）或替换（`REPLACE`）模式。
  + 请求/响应：`LoadToCandidateRequest`/`LoadToCandidateResponse`
  + 关键参数：
    * `candidate_id`
    * `type`：加载模式（`MERGE`或`REPLACE`）
    * `config`：待加载的配置数据（`DataTree`格式）
- **Commit**
  + 功能：提交候选配置，支持两阶段提交（验证、准备、应用、中止等阶段）。
  + 请求/响应：`CommitRequest`/`CommitResponse`
  + 关键参数：
    * `candidate_id`
    * `phase`：提交阶段（`VALIDATE`/`PREPARE`/`APPLY`/`ABORT`/`ALL`）
    * `comment`：提交注释
  + 响应字段：
    * `transaction_id`：事务ID
    * `error_message`：错误信息（若提交失败）
- **ListTransactions**
  + 功能：列出所有历史事务的元数据（事务ID、客户端、时间、注释）。
  + 请求/响应：`ListTransactionsRequest`/`ListTransactionsResponse`
  + 响应字段：事务列表（流式返回）
- **GetTransaction**
  + 功能：获取指定事务ID对应的详细配置数据。
  + 请求/响应：`GetTransactionRequest`/`GetTransactionResponse`
  + 关键参数：`transaction_id`
  + 响应字段：`config`（事务对应的配置数据）
- **LockConfig**
  + 功能：锁定运行配置，防止其他客户端修改。
  + 请求/响应：`LockConfigRequest`/`LockConfigResponse`
- **UnlockConfig**
  + 功能：解锁运行配置。
  + 请求/响应：`UnlockConfigRequest`/`UnlockConfigResponse`
- **Execute**
  + 功能：执行自定义的YANG RPC操作（如触发路由重计算、清除接口统计等）。
  + 请求/响应：`ExecuteRequest`/`ExecuteResponse`
  + 关键参数：
    * `path`：RPC的YANG路径
    * `input`：输入参数（`PathValue`列表）
  + 响应字段：`output`（RPC执行结果）

### 关键数据结构

#### 枚举类型

- **DataType​**（数据类别）
  + `CONFIG`：仅配置数据
  + `STATE`：仅状态数据
  + `ALL`：配置 + 状态数据
- ​**LoadType**​（加载模式）
  + `MERGE`：合并到现有配置
  + `REPLACE`：替换现有配置
- ​**CommitPhase**​（提交阶段）
  + `VALIDATE`：验证配置有效性
  + `PREPARE`：预提交检查
  + `APPLY`：应用配置
  + `ABORT`：中止事务
  + `ALL`：完整提交流程
- ​**Encoding**​（编码格式）
  + `JSON`：JSON 格式
  + `XML`：XML 格式

#### 数据结构
- ​**DataTree**：树形结构，包含多个 `PathValue` 节点
- ​**PathValue**：路径-值对，表示配置项
- ​**ModuleData**：模块元数据（名称、版本、组织等）
- ​**TransactionMetadata**：事务 ID、时间戳、操作用户等信息

### 典型使用场景
1. **​配置批量更新**
创建候选配置 → 通过 `EditCandidate` 或 `LoadToCandidate` 修改 → 验证并提交 (`Commit`)。

2. **​安全回滚**
使用 `ListTransactions` 查找历史配置 → 通过 `GetTransaction` 获取旧配置 → 加载到候选配置并提交。

3. **​并发控制**
修改前调用 `LockConfig` → 修改配置 → 提交后调用 `UnlockConfig`。

4. **​状态监控**
定期调用 `Get` 方法（`type=STATE`）获取实时状态数据。


****

本文参考

> 1. [FRR北向gRPC](https://docs.frrouting.org/projects/dev-guide/en/latest/grpc.html)
> 2. [Frrouting快速入门——北向接口事务cli与gRPC(二)](https://blog.csdn.net/puhaiyang/article/details/140449617)