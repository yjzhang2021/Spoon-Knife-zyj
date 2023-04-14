
# 安全算子服务接口列表
1. 算子服务化接口
2. 算子表达式查询请求
3. 异步化关闭接口
4. 异步化辅助查询接口

# 1 算子服务化接口


## 接口描述
算子服务化主接口，执行一个表达式并定义输出结果

POST /secure_operate/v1/execute
## 请求体(Request Body)

| 参数名称 | 数据类型  | 可选 | 描述 |
| :-----  | :---          | :----: | :------ |
| taskId | string | false | 任务ID，用于约定对齐算子、算法上下文的任务编号|
| subTaskId | string | true | [可选]子任务ID，用于算法对任务进行拆分或并行计算时使用，一般编码方式是taskId+分片后缀。|
| token | string | false | 控制面下发的Token，用于多方之间处理权限核验|
| asyncMode | boolean | true | [可选]同步或异步模式，默认为false，false表示同步调用，同步情况下，算子服务需要在计算任务完成之后返回结果，上层算法阻塞等待。true表示采用异步调用，异步调用情况下，算子服务在受理服务请求后，即可返回，算法层会根据outputMethod约定的存储位置异步获取结果。
| timeout | int32 | true | [可选]超时时间，单位为秒，默认为0表示不设置超时时间。算子服务如果超时，会强制结束任务，并返回错误。|
| mpcProtocol | object | false | 多方安全计算协议名称，用于约定算子实现的技术路线。|
| ⇥ protocolCode | string | false | 协议名称编码|
| ⇥ providerCode | string | false | 提供厂商名称编码|
| ⇥ version | string | false | 协议版本号|
| ⇥ param | object | false | 协议超参|
| expression | string | false | 用于描述算子计算操作，比如mvm表示矩阵向量乘法，matmul表示矩阵乘法；|
| parties | array[string] | false | 参与方ID列表，由发起方（或协调方）制定，该列表中个参与方的位置在一次计算任务重，是固定的，各参与方不允许对其调整，因为在计算过程|中，算子需要根据该参与方序列保障安全算子协议的协同次序。
| localPartyId | string | false | 本方的参与方ID|
| resultParties | array[string] | false | 结果获取方列表，由发起方（或协调方）进行设置。|
| inputs | array[object] | false | 数据输入|
| ⇥ dataValueTag | object | false | 对一个算子输入、输出参数的描述信息|
| ⇥⇥ type | string | false | 标记数据传输方式，Direct,标识直接传值，如果希望使用存储引擎来传值，可以使用引擎名称，例如myRedis。|
| ⇥⇥ name | string | false | 与表达式中的对应参数name 进行关联， 让表达式能够根据name 定位的数据。|
| ⇥⇥ uri | string | false | 非 Direct 情况下使用，使用存储引擎时， 指定存储引擎的资源定位符|
| ⇥⇥ key | string | false | 非 Direct 情况下使用，使用存储引擎时，使用的 key 用于数据读写。|
| ⇥⇥ dtype | string | false | 用于标记输入输出数据的类型，在数据传输过程中，数据块以字节数组的形式进行传输，dtype对数据块的格式进行描述，用于将字节数据转换为所需要的数据类型，详细说明，参考dtype枚举说明。
| ⇥⇥ shape | array[int32] | false | 用于标记输入输出数据的形状，shape的元素给出了相应维度上的数组尺寸的长度。|
| ⇥⇥ delete | boolean | false | 默认为false,数据使用完成之后，是否清理。|
| ⇥ directValue | string | false | 直接传值的数据，如果不采集直接传值的方式，则该值为空。该字段的处理方式上，是将要传输的数据块的转换为字节数组，再通过Base64编码后|得到字符串形式的值，以便支持不同类型的RPC协议。数据块转换到字节数组的方式，由DataValueTag 中的dtype和shape确定。
| outputMethod | array[object] | false | 数据返回方式|
| ⇥ type | string | false | 标记数据传输方式，Direct,标识直接传值，如果希望使用存储引擎来传值，可以使用引擎名称，例如myRedis。|
| ⇥ name | string | false | 与表达式中的对应参数name 进行关联， 让表达式能够根据name 定位的数据。|
| ⇥ uri | string | false | 非 Direct 情况下使用，使用存储引擎时， 指定存储引擎的资源定位符|
| ⇥ key | string | false | 非 Direct 情况下使用，使用存储引擎时，使用的 key 用于数据读写。|
| ⇥ dtype | string | false | 用于标记输入输出数据的类型，在数据传输过程中，数据块以字节数组的形式进行传输，dtype对数据块的格式进行描述，用于将字节数据转换为所需要的数据类型，详细说明，参考dtype枚举说明。|
| ⇥ shape | array[int32] | false | 用于标记输入输出数据的形状，shape的元素给出了相应维度上的数组尺寸的长度。|
| ⇥ delete | boolean | false | 默认为false,数据使用完成之后，是否清理。|
| phases | string | true | [可选] 有状态服务的执行阶段，缺省为无状态服务，可分阶段执行，可指定一个或多个阶段执行，定义为三个阶段，PREPAR（准备阶段）、CALCULATE（计算阶段）,RETRIEVE（恢复阶段）

## 响应体

200 响应数据格式：JSON

| 参数名称 | 数据类型  | 可选 | 描述 |
| :-----  | :---          | :----: | :------ |
| code | int32 | false | 0 正常，否则错误码 错误码分段: 100-199表示系统层面的异常，比如算子服务未正常启动； 200-299表示依赖本地环境所产生的异常，如依赖的节点内存储系统发生异常； 300-399表示节点间的处理异常，包括节点间通信和节点间的系统异常； 400-999 表示内部的逻辑错误，比如参数错误产生的异常，数据格式错误、表达式不存在、算子异常终止等。|
| msg | string | false | 返回信息描述|
| result | array[object] | false | 本次计算返回的结果，如果是异步调用模式下，则不立即返回此参数，可通过异步结果查询接口，获取此参数。|
| ⇥ dataValueTag | object | false | 对算子输入输出一个参数的描述。|
| ⇥⇥ type | string | false | 标记数据传输方式，Direct,标识直接传值，如果希望使用存储引擎来传值，可以使用引擎名称，例如myRedis。|
| ⇥⇥ name | string | false | 与表达式中的对应参数name 进行关联， 让表达式能够根据name 定位的数据。|
| ⇥⇥ uri | string | false | 非 Direct 情况下使用，使用存储引擎时， 指定存储引擎的资源定位符|
| ⇥⇥ key | string | false | 非 Direct 情况下使用，使用存储引擎时，使用的 key 用于数据读写。|
| ⇥⇥ dtype | string | false | 用于标记输入输出数据的类型，在数据传输过程中，数据块以字节数组的形式进行传输，dtype对数据块的格式进行描述，用于将字节数据转换为所需要的数据类型，详细说明，参考dtype枚举说明。|
| ⇥⇥ shape | array[int32] | false | 用于标记输入输出数据的形状，shape的元素给出了相应维度上的数组尺寸的长度。|
| ⇥⇥ delete | boolean | false | 默认为false,数据使用完成之后，是否清理。|
| ⇥ directValue | string | false | 直接传值的数据，如果不采集直接传值的方式，则该值为空。该字段的处理方式上，是将要传输的数据块的转换为字节数组，再通过Base64编码后得到字符串形式的值，以便支持不同类型的RPC协议。数据块转换到字节数组的方式，由DataValueTag 中的dtype和shape确定。|


# 2 算子表达式查询请求



# 接口描述

算子表达式查询请求， 用于查询当前算子服务支持的表达式

POST /secure_operate/v1/expressions

## 响应体

● 200 响应数据格式：JSON

| 参数名称 | 数据类型  | 可选 | 描述 |
| :-----  | :---          | :----: | :------ |
| expressions | array[object] | false | 表达式信息数组 |
| ⇥ protocolCode | string | false | 协议名称编码 |
| ⇥ providerCode | string | false | 提供厂商名称编码 |
| ⇥ version | string | false | 协议版本号 |
| ⇥ expression | string | false | 表达式名称 |
| ⇥ params | array[object] | false | 表达式超参 |
| ⇥⇥ paramKey | string | false | 超参的形参编码 |
| ⇥⇥ type | string | false | 参数的数据类型，限于基本数据类型。 |
| ⇥⇥ comment | string | false | 超参说明 |
| ⇥⇥ paramDemo | string | false | 参数实例 |

# 3 异步化关闭接口


## 接口描述

异步化关闭接口,用于关闭一个已发起的异步任务

POST /secure_operate/v1/kill

## 请求体(Request Body)

| 参数名称 | 数据类型  | 可选 | 描述 |
| :-----  | :---          | :----: | :------ |
| taskId | string | false | 任务ID，用于约定对齐算子、算法上下文的任务编号 |
| subTaskId | string | true | [可选]子任务ID，用于算法对任务进行拆分或并行计算时使用，一般编码方式是taskId+分片后缀。 |
| localPartyId | string | false | 本方的参与方ID |

## 响应体

● 200 响应数据格式：JSON

| 参数名称 | 数据类型  | 可选 | 描述 |
| :-----  | :---          | :----: | :------ |
|code | int32 | false | 0 正常，否则错误码 错误码分段: 100-199表示系统层面的异常，比如算子服务未正常启动；  |200-299表示依赖本地环境所产生的异常，如依赖的节点内存储系统发生异常； 300-399表示节点间的处理异常，包括节点间通信和节点间的系统异常； 400-999 表示内部的逻辑错误，比如参数错误产生的异常，数据格式错误、表达式不存在、算子异常终止等。
|msg | string | false | 返回信息描述 |
|result | array[object] | false | 本次计算返回的结果，如果是异步调用模式下，则不立即返回此参数，可通过异步结果查询接口，获取此参数。 |
|⇥ dataValueTag | object | false | 对算子输入输出一个参数的描述。 |
|⇥⇥ type | string | false | 标记数据传输方式，Direct,标识直接传值，如果希望使用存储引擎来传值，可以使用引擎名称，例如myRedis。 |
|⇥⇥ name | string | false | 与表达式中的对应参数name 进行关联， 让表达式能够根据name 定位的数据。 |
|⇥⇥ uri | string | false | 非 Direct 情况下使用，使用存储引擎时， 指定存储引擎的资源定位符 |
|⇥⇥ key | string | false | 非 Direct 情况下使用，使用存储引擎时，使用的 key 用于数据读写。 |
|⇥⇥ dtype | string | false |  |用于标记输入输出数据的类型，在数据传输过程中，数据块以字节数组的形式进行传输，dtype对数据块的格式进行描述，用于将字节数据转换为所需要的数据类型，详细说明，参考dtype枚举说明。
|⇥⇥ shape | array[int32] | false | 用于标记输入输出数据的形状，shape的元素给出了相应维度上的数组尺寸的长度。 |
|⇥⇥ delete | boolean | false | 默认为false,数据使用完成之后，是否清理。 |
|⇥ directValue | string | false | 直接传值的数据，如果不采集直接传值的方式，则该值为空。该字段的处理方式上，是将要传输的数据块的转换为字节数组，再通过Base64编码后得到字 |符串形式的值，以便支持不同类型的RPC协议。数据块转换到字节数组的方式，由DataValueTag 中的dtype和shape确定。


# 4 异步化辅助查询接口
## 接口描述

异步化辅助查询接口，用于查询一个已发起的异步任务的执行结果

POST /secure_operate/v1/query

## 请求体(Request Body)

| 参数名称 | 数据类型  | 可选 | 描述 |
| :-----  | :---          | :----: | :------ |
| taskId | string | false | 任务ID，用于约定对齐算子、算法上下文的任务编号 |
| subTaskId | string | false | [可选]子任务ID，用于算法对任务进行拆分或并行计算时使用，一般编码方式是taskId+分片后缀。 |
| localPartyId | string | false | 本方的参与方ID |

## 响应体

● 200 响应数据格式：JSON

| 参数名称 | 数据类型  | 可选 | 描述 |
| :-----  | :---          | :----: | :------ |
| code | int32 | false | 0 正常，否则错误码 错误码分段: 100-199表示系统层面的异常，比如算子服务未正常启动； 200-299表示依赖本地环境所产生的异常，如依赖的节点内存储系统发生异常； 300-399表示节点间的处理异常，包括节点间通信和节点间的系统异常； 400-999 表示内部的逻辑错误，比如参数错误产生的异常，数据格式错误、表达式不存在、算子异常终止等。 |
| msg | string | false | 返回信息描述 |
| result | array[object] | false | 本次计算返回的结果，如果是异步调用模式下，则不立即返回此参数，可通过异步结果查询接口，获取此参数。 |
| ⇥ dataValueTag | object | false | 对算子输入输出一个参数的描述。 |
| ⇥⇥ type | string | false | 标记数据传输方式，Direct,标识直接传值，如果希望使用存储引擎来传值，可以使用引擎名称，例如myRedis。 |
| ⇥⇥ name | string | false | 与表达式中的对应参数name 进行关联， 让表达式能够根据name 定位的数据。 |
| ⇥⇥ uri | string | false | 非 Direct 情况下使用，使用存储引擎时， 指定存储引擎的资源定位符 |
| ⇥⇥ key | string | false | 非 Direct 情况下使用，使用存储引擎时，使用的 key 用于数据读写。 |
| ⇥⇥ dtype | string | false | 用于标记输入输出数据的类型，在数据传输过程中，数据块以字节数组的形式进行传输，dtype对数据块的格式进行描述，用于将字节数据转换为所需要的数据类型，详细说明，参考dtype枚举说明。 |
| ⇥⇥ shape | array[int32] | false | 用于标记输入输出数据的形状，shape的元素给出了相应维度上的数组尺寸的长度。 |
| ⇥⇥ delete | boolean | false | 默认为false,数据使用完成之后，是否清理。 |
| ⇥ directValue | string | false | 直接传值的数据，如果不采集直接传值的方式，则该值为空。该字段的处理方式上，是将要传输的数据块的转换为字节数组，再通过Base64编码后得到字符串形式的值，以便支持不同类型的RPC协议。数据块转换到字节数组的方式，由DataValueTag 中的dtype和shape确定。 |

