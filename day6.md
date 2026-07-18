# 学习记录  
### 记录人：  
**邹晓南**  
### 学习日期：  
**7.18**  
### 学习内容：  
- [x] **ros2入门基础知识之通信接口**    
- [x] **ros2入门基础知识之服务**    

## 一、自定义接口IDL  
### 1.1 三大通信接口基础
***统一使用 IDL 定义，自动生成 C++/Python 代码***
| 文件后缀 | 通信方式 | 结构 |  
| --------| --------|------|  
|`.msg`   | Topic 单向发布订阅，无应答 | 字段 + 常量 |  
|`.srv`   | 同步客户端 / 服务端请求应答 | 请求 `---` 响应 |  
|`.action` | 长任务异步通信，可取消 + 实时反馈 | `Goal --- Result --- Feedback` |  

**通用规则：**
1. 文件分别存放于包内 msg/ srv/ action/
2. 字段、常量语法三类接口完全互通
3. 自定义接口必须配置`package.xml` + `CMakeLists.txt` 编译
### 1.2 msg 消息 IDL 语法
**1.2.1 基础格式**
```
# 字段：类型 变量名 [默认值]
uint8 x 42
string name "test"

# 常量：类型 全大写常量=值
uint8 TYPE_HOME=0
```
**1.2.2 数据类型**
1. 基础内置类型：`bool` `int8/uint8~int64/uint64` `float32` `float64` `string`
2. 复合类型：`包名/消息名`，同包消息可省略包名
3. 数组写法  
```
int32[] arr        # 无界动态数组（常用）
int32[5] arr       # 固定长度数组
int32[<=5] arr     # 有界最大长度数组
string<=10 str     # 限长字符串
string<=10[<=5] list # 限长字符串数组
```
**1.2.3 命名与限制**
1. 字段：小写字母 + 下划线，首字符字母，不可连续下划线、不可下划线结尾
2. 常量：必须全大写，代码中 `消息实例.常量名` 访问
3. 默认值限制：字符串数组、嵌套复杂消息不支持默认值；字符串需引号包裹
### 1.3 自定义 msg 功能包完整实操
**1.3.1 创建包与目录**  
```
ros2 pkg create --build-type ament_cmake --license Apache-2.0 more_interfaces
mkdir more_interfaces/msg
```
**1.3.2 示例 msg 文件 `msg/AddressBook.msg`**
```
uint8 PHONE_TYPE_HOME=0
uint8 PHONE_TYPE_WORK=1
uint8 PHONE_TYPE_MOBILE=2

string first_name
string last_name
string phone_number
uint8 phone_type
```
**1.3.3 package.xml 必加依赖**
```
<buildtool_depend>rosidl_default_generators</buildtool_depend>
<exec_depend>rosidl_default_runtime</exec_depend>
<member_of_group>rosidl_interface_packages</member_of_group>
```
**1.3.4 CMakeLists 消息生成基础配置
cmake**
```
find_package(rosidl_default_generators REQUIRED)
set(msg_files "msg/AddressBook.msg")
rosidl_generate_interfaces(${PROJECT_NAME} ${msg_files})
ament_export_dependencies(rosidl_default_runtime)
```
**1.3.5 C++ 发布节点核心配置**
1. 引入头文件：`#include "more_interfaces/msg/address_book.hpp"`
2. 编译可执行文件
cmake
```
find_package(rclcpp REQUIRED)
add_executable(publish_address_book src/publish_address_book.cpp)
ament_target_dependencies(publish_address_book rclcpp)
install(TARGETS publish_address_book DESTINATION lib/${PROJECT_NAME})
```
3. 同包消息专属链接（跨包无需，漏写报错）
cmake
```
rosidl_get_typesupport_target(cpp_typesupport_target ${PROJECT_NAME} rosidl_typesupport_cpp)
target_link_libraries(publish_address_book "${cpp_typesupport_target}")
```
3.6 编译运行指令
```
# 编译
cd ~/ros2_ws
colcon build --packages-up-to more_interfaces
# 刷新环境（新开终端必执行）
source install/local_setup.bash
# 运行节点
ros2 run more_interfaces publish_address_book
# 验证话题
ros2 topic echo /address_book
```
### 1.4 srv /action 语法规范
**1.4.1 .srv 服务（srv/xxx.srv）**
`---` 分割请求、响应
idl
```
# 请求
string input
---
# 响应
bool success
```
规则：不可嵌套服务，请求 / 响应字段可为空
**1.4.2 .action 动作（action/xxx.action）**
两条 `---` 分割三段
idl
```
# Goal
int32 order
---
# Result
int32[] sequence
---
# Feedback
int32[] sequence
```
### 1.5 跨包引用外部消息
**1.5.1 msg 中声明外部类型**
idl  
`外部包名/消息名[] address_list`  
**1.5.2 package.xml 添加依赖**
xml
```
<build_depend>外部包名</build_depend>
<exec_depend>外部包名</exec_depend>
```
**1.5.3 CMakeLists 补充依赖**
cmake
```
find_package(外部包名 REQUIRED)
rosidl_generate_interfaces(${PROJECT_NAME} ${msg_files} DEPENDENCIES 外部包名)  
```  
### 1.6 总结  
1. IDL 统一规范三类接口，学会 msg 后 srv/action 一通百通
2. 自定义消息三要素：msg 定义文件 + xml 依赖 + CMake 消息生成
3. 消息与节点同包必须添加 typesupport 链接代码
4. 新开终端必须 source 环境，否则找不到消息 / 可执行文件
5. 跨包使用外部接口需在 CMake 声明 DEPENDENCIES 依赖  

## 二、动作  
### 2.1 Action 核心基础概念（必记）
**2.1.1 定位：**  
专门用于长时间运行任务的通信方式，基于话题 + 服务封装实现
**2.1.2 和 Service（服务）核心区别**
| 特性| action动作 |Service服务 |  
| ----|-----------|-----------|  
|执行时长|长耗时任务|瞬时短任务  |  
|中途取消  |支持客户端取消目标|不支持中断|  
|持续实时反馈  | 持续实时反馈|仅执行完成返回单次结果|  
|通信模型|Action Client ↔ Action Server|Client ↔ Server|

**2.1.3 Action 文件 .action 三段固定结构（两条---分割）**
```idl   
# Goal 目标：客户端下发任务参数
float32 theta
---
# Result 结果：任务完成后最终返回数据
float32 delta
---
# Feedback 反馈：任务执行中持续推送中间数据
float32 remaining
```
**2.1.4 典型使用场景：**  
机器人导航、机械臂轨迹运动、小车定点行走等耗时任务
### 2.2 turtlesim 实操演示流程
**2.2.1 启动环境两个节点**
```bash
# 终端1：海龟仿真（内置Action Server）
ros2 run turtlesim turtlesim_node
# 终端2：键盘控制（内置Action Client）
ros2 run turtlesim turtle_teleop_key
```
• ***方向键*** :话题 /turtle1/cmd_vel 控制移动  
• ***G/B/V/C/D/E/R/T***：发送 Action 目标，海龟旋转至指定角度
• ***F***：取消当前正在执行的旋转目标
**2.2.2 两种终止任务情况** 
1. 客户端主动取消：按 F，日志输出 `Rotation goal canceled`
2. 服务端主动终止（abort）：下发新目标时旧任务未完成，日志 `Aborting previous goal`
注意：不同 Action 服务端处理新目标逻辑不统一，不一定都会放弃旧任务
### 2.3 Action 全套命令行工具（实操必考）
**2.3.1 查看节点包含的 Action 服务端 / 客户端**
```bash
ros2 node info /节点名
```
输出分区：`Action Servers`（服务端，接收目标）、`Action Clients`（客户端，发送目标）
示例：
• `/turtlesim`：Action Server `/turtle1/rotate_absolute`
• `/teleop_turtle`：Action Client `/turtle1/rotate_absolute`
**2.3.2 列出系统中所有 Action**
```bash
# 只显示action名称
ros2 action list
# 显示action名称 + 完整类型（最常用）
ros2 action list 
-t
# 输出示例：/turtle1/rotate_absolute [turtlesim/action/RotateAbsolute]
```
**2.3.3 查看某个 Action 的客户端 / 服务端数量及对应节点**
```bash
ros2 action info /action名称
# 示例：ros2 action info /turtle1/rotate_absolute
# 输出客户端数量、服务端数量、对应节点名
```
**2.3.4 查看 Action 接口定义（三段 Goal/Result/Feedback**）
```bash
ros2 interface show 包名/action/动作类型
# 示例
ros2 interface show turtlesim/action/RotateAbsolute
```
**2.3.5 命令行手动发送 Action 目标（核心实操命令）**
基础语法：
```bash
ros2 action send_goal <action名> <action类型> <YAML格式参数>
```
1. 普通发送（只看最终结果，无过程反馈）
```bash
ros2 action send_goal /turtle1/rotate_absolute turtlesim/action/RotateAbsolute "{theta: 1.57}"
```  

2. 带实时反馈输出（添加--feedback）
```bash
ros2 action send_goal /turtle1/rotate_absolute turtlesim/action/RotateAbsolute "{theta: -1.57}" --feedback
```
• 每个目标拥有唯一 ID
• Feedback 持续打印中间状态，任务结束输出最终 Result 与执行状态`SUCCEEDED`
### 2.4 核心总结
1. Action 三大核心优势：支持长任务、可中途取消、执行过程持续反馈；
2. `.action` 文件三段式结构：Goal (目标) --- Result (最终结果) --- Feedback (过程反馈)；
3. 一套完整工具链：`node info`/`action list`/`action info`/`interface show`/`send_goal`；
4. 通信两端：Action Client 发送目标，Action Server 执行任务、返回反馈与结果；
5. 机器人开发标准场景：导航、机械臂运动等耗时任务优先使用 Action