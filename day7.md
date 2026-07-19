# 学习记录  
### 记录人：  
**邹晓南**  
### 学习日期：  
**7.19**  
### 学习内容：  
- [x] **ros2入门基础知识之参数**    
- [x] **ros2入门基础知识之Launch**    

## 一、参数  
### 1.1 ROS2 Param 七大核心命令（重中之重）
***统一通用前缀：ros2 param + 子命令***
**1.1.1 `ros2 param list` —— 查看所有参数名**
turtlesim 关键参数解析：
- use_sim_time：是否启用仿真时间
- qos_overrides：通信 QOS 配置（只读参数）  

**1.1.2 `ros2 param get` —— 查看参数值**
格式：
`ros2 param get <节点名> <参数名>`
示例：  
```
# 查看背景绿色值
ros2 param get /turtlesim background_g
# 默认输出：Integer value is: 86

# 其余两个颜色默认值
ros2 param get /turtlesim background_r  # 69
ros2 param get /turtlesim background_b  # 255  
```  

**1.1.3 `ros2 param describe`查看参数的类型、用途、是否可读**  
格式：
`ros2 param describe <节点名> <参数名>`
示例：查看仿真时间参数详情
`ros2 param describe /turtlesim use_sim_time`  

**1.1.4 `ros2 param set` —— 运行时动态改参数**  
格式：
`ros2 param set <节点名> <参数名> <新值>`
示例：修改窗口红色通道，改变背景色
`ros2 param set /turtlesim background_r 150`
*关键特点：临时生效，仅当前会话有效，重启节点后恢复默认，无法永久保存。*

**1.1.5 `ros2 param dump` —— 导出/保存参数配置**  
格式：
`ros2 param dump <节点名> > 保存文件名.yaml`
示例：
`ros2 param dump /turtlesim > turtlesim.yaml`
导出的文件包含：可读写参数 + 只读参数的完整配置，可用于后续恢复配置。

**1.1.6 `ros2 param load` —— 运行时加载参数文件**  
格式：
`ros2 param load <节点名> <参数文件.yaml>`
示例：
`ros2 param load /turtlesim turtlesim.yaml`
注意事项：
- 普通可读写参数（背景色、use_sim_time）可正常加载
- qos_overrides 类只读参数运行时无法修改，加载失败属于正常现象，非报错  

**1.1.7 节点启动时加载参数文件**  
格式：
`ros2 run <包名> <可执行文件> --ros-args --params-file <参数文件.yaml>`
示例：
`ros2 run turtlesim turtlesim_node --ros-args --params-file turtlesim.yaml`
*核心特性：启动阶段加载参数，所有参数（含只读参数）均可生效，实现配置永久固化。*    

### 1.2 参数设置程序代码  
```
import rclpy                                     # ROS2 Python接口库    
from rclpy.node import Node                    # ROS2 节点类

class ParameterNode(Node):
    def __init__(self, name):
        super().__init__(name)                                    # ROS2节点父类初始化
        self.timer = self.create_timer(2, self.timer_callback)    # 创建一个定时器（单位为秒的周期，定时执行的回调函数）
        self.declare_parameter('robot_name', 'mbot')        # 创建一个参数，并设置参数的默认值

    def timer_callback(self):                             # 创建定时器周期执行的回调函数
        robot_name_param = self.get_parameter('robot_name').get_parameter_value().string_value   # 从ROS2系统中读取参数的值

        self.get_logger().info('Hello %s!' % robot_name_param)     # 输出日志信息，打印读取到的参数值

        new_name_param = rclpy.parameter.Parameter('robot_name',   # 重新将参数值设置为指定值
                            rclpy.Parameter.Type.STRING, 'mbot')
        all_new_parameters = [new_name_param]
        self.set_parameters(all_new_parameters)                    # 将重新创建的参数列表发送给ROS2系统

def main(args=None):                                 # ROS2节点主入口main函数
    rclpy.init(args=args)                            # ROS2 Python接口初始化
    node = ParameterNode("param_declare")            # 创建ROS2节点对象并进行初始化
    rclpy.spin(node)                                 # 循环等待ROS2退出
    node.destroy_node()                              # 销毁节点对象
    rclpy.shutdown()                                 # 关闭ROS2 Python接口  
```  

## 二、Launch文件  
### 2.1 是什么（定义）
- Launch文件是ROS2中一次性启动多个节点、设置参数、命名空间、重映射的工具。
- ROS2的Launch文件本质是Python脚本（.py）

### 2.2 为什么用
1. 避免每个终端单独ros2 run，提高效率。  
2. 管理复杂系统中的节点依赖、参数配置、话题重命名。  
3. 支持按条件启动（如根据环境变量或硬件是否存在）。  
4. 便于复用（include其他launch文件）。  

### 2.3 核心组件（必须掌握）
| 组件|	作用 | 对应函数/类|  
|---- | ------| -------- |  
|Action| 启动一个“动作”，最常见的是启动节点 |	Node() |
|Node | 描述要启动的节点：包名、可执行文件名、参数、命名空间、重映射等 | Node(package, executable, ...) |
| Arguments | 让launch文件支持外部传入参数（如 --ros-args 或 --arguments）|	DeclareLaunchArgument() + LaunchConfiguration() |  
|Include	|嵌套其他launch文件，实现模块化	|IncludeLaunchDescription()|
|Group	|给一组节点统一设置命名空间或条件	|Group(action=[...], namespace='...')|
|Condition	|条件执行（如 IfCondition / UnlessCondition）|	IfCondition(LaunchConfiguration('enable'))|  

### 2.4. 典型写法（骨架代码）
```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(
            package='my_pkg',
            executable='my_node',
            name='my_node_instance',
            namespace='ns1',
            parameters=[{'param1': 'value1'}, '/path/to/params.yaml'],
            remappings=[('chatter', 'my_chatter')],
            output='screen',   # 打印到终端
            emulate_tty=True   # 让日志及时显示
        ),
        # 可以添加更多Node或其它Action
    ])  
```
### 2.5 常用技巧
- **参数传递：**  
1. 通过 `parameters=` 传入dict或yaml文件。
2. 获取外部参数：`DeclareLaunchArgument('arg_name') `+`LaunchConfiguration('arg_name')`。  
```  
import os

from ament_index_python.packages import get_package_share_directory  # 查询功能包路径的方法

from launch import LaunchDescription   # launch文件的描述类
from launch_ros.actions import Node    # 节点启动的描述类


def generate_launch_description():     # 自动生成launch文件的函数
   config = os.path.join(              # 找到参数文件的完整路径
      get_package_share_directory('learning_launch'),
      'config',
      'turtlesim.yaml'
      )

   return LaunchDescription([          # 返回launch文件的描述信息
      Node(                            # 配置一个节点的启动
         package='turtlesim',          # 节点所在的功能包
         executable='turtlesim_node',  # 节点的可执行文件名
         namespace='turtlesim2',       # 节点所在的命名空间
         name='sim',                   # 对节点重新命名
         parameters=[config]           # 加载参数文件
      )
   ])  
```

```  
import os

from ament_index_python.packages import get_package_share_directory # 查询功能包路径的方法

from launch import LaunchDescription    # launch文件的描述类
from launch_ros.actions import Node     # 节点启动的描述类


def generate_launch_description():      # 自动生成launch文件的函数
   rviz_config = os.path.join(          # 找到配置文件的完整路径
      get_package_share_directory('learning_launch'),
      'rviz',
      'turtle_rviz.rviz'
      )

   return LaunchDescription([           # 返回launch文件的描述信息
      Node(                             # 配置一个节点的启动
         package='rviz2',               # 节点所在的功能包
         executable='rviz2',            # 节点的可执行文件名
         name='rviz2',                  # 对节点重新命名
         arguments=['-d', rviz_config]  # 加载命令行参数
      )
   ])
```

- **重映射（remap）：**  
解决话题名冲突，格式 `[('原始话题', '目标话题')]`。

- **命名空间：**  
在Node中设置 `namespace`，或使用 `Group` 批量设置。
```  
from launch import LaunchDescription      # launch文件的描述类
from launch_ros.actions import Node       # 节点启动的描述类

def generate_launch_description():        # 自动生成launch文件的函数
    return LaunchDescription([            # 返回launch文件的描述信息
        Node(                             # 配置一个节点的启动
            package='turtlesim',          # 节点所在的功能包
            namespace='turtlesim1',       # 节点所在的命名空间
            executable='turtlesim_node',  # 节点的可执行文件名
            name='sim'                    # 对节点重新命名
        ),
        Node(                             # 配置一个节点的启动
            package='turtlesim',          # 节点所在的功能包
            namespace='turtlesim2',       # 节点所在的命名空间
            executable='turtlesim_node',  # 节点的可执行文件名
            name='sim'                    # 对节点重新命名
        ),
        Node(                             # 配置一个节点的启动
            package='turtlesim',          # 节点所在的功能包
            executable='mimic',           # 节点的可执行文件名
            name='mimic',                 # 对节点重新命名
            remappings=[                  # 资源重映射列表
                ('/input/pose', '/turtlesim1/turtle1/pose'),         # 将/input/pose话题名修改为/turtlesim1/turtle1/pose
                ('/output/cmd_vel', '/turtlesim2/turtle1/cmd_vel'),  # 将/output/cmd_vel话题名修改为/turtlesim2/turtle1/cmd_vel
            ]
        )
    ])
```
- **条件启动：**  
结合 `IfCondition`，例如调试模式才启动可视化节点。

- **包含其他launch：**  
用 `IncludeLaunchDescription` 实现分层启动。
```  
import os

from ament_index_python.packages import get_package_share_directory  # 查询功能包路径的方法

from launch import LaunchDescription                 # launch文件的描述类
from launch.actions import IncludeLaunchDescription  # 节点启动的描述类
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch.actions import GroupAction               # launch文件中的执行动作
from launch_ros.actions import PushRosNamespace      # ROS命名空间配置

def generate_launch_description():                   # 自动生成launch文件的函数
   parameter_yaml = IncludeLaunchDescription(        # 包含指定路径下的另外一个launch文件
      PythonLaunchDescriptionSource([os.path.join(
         get_package_share_directory('learning_launch'), 'launch'),
         '/parameters_nonamespace.launch.py'])
      )
  
   parameter_yaml_with_namespace = GroupAction(      # 对指定launch文件中启动的功能加上命名空间
      actions=[
         PushRosNamespace('turtlesim2'),
         parameter_yaml]
      )

   return LaunchDescription([                        # 返回launch文件的描述信息
      parameter_yaml_with_namespace
   ])
```
### 2.6  执行方式
```bash
# 基础启动
ros2 launch my_pkg my_launch.py

# 带外部参数（需在launch中先声明）
ros2 launch my_pkg my_launch.py arg_name:=arg_value

# 查看launch文件中的参数列表
ros2 launch -s my_pkg my_launch.py  # 或使用 --show-args
```  
