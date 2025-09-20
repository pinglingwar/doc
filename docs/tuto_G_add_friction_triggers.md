# 如何添加摩擦触发器

[摩擦触发器](https://github.com/carla-simulator/carla/pull/1562) (*Friction Triggers*) 是可以在运行时添加的框触发器，允许用户定义车辆车轮在此类触发器内时的不同摩擦力。例如，这对于动态地在地图的某些区域中制作光滑的表面可能很有用。

为了使用 PythonAPI 生成摩擦触发器，用户必须首先获取
`static.trigger.friction` 蓝图定义，然后为该蓝图定义设置以下必要属性：

- *friction*: 当车辆位于触发盒内时，触发盒的摩擦力。
- *extent_x*: X 坐标中边界框的范围（以厘米为单位）。
- *extent_y*: Y 坐标中边界框的范围（以厘米为单位）。
- *extent_z*: Z 坐标中边界框的范围（以厘米为单位）。

完成此操作后，定义一个变换来指定摩擦触发器的位置和旋转并生成它。

##### 例子

```py
import carla

def main():
    # 连接到客户端
    client = carla.Client('127.0.0.1', 2000)
    client.set_timeout(2.0)

    # 获取事件和参与者
    world = client.get_world()
    actors = world.get_actors()

    # 找到摩擦触发器的蓝图
    friction_bp = world.get_blueprint_library().find('static.trigger.friction')

    extent = carla.Location(700.0, 700.0, 700.0)

    friction_bp.set_attribute('friction', str(0.0))
    friction_bp.set_attribute('extent_x', str(extent.x))
    friction_bp.set_attribute('extent_y', str(extent.y))
    friction_bp.set_attribute('extent_z', str(extent.z))

    # 生成摩擦触发器
    transform = carla.Transform()
    transform.location = carla.Location(100.0, 0.0, 0.0)
    world.spawn_actor(friction_bp, transform)

    # 可视化触发器（可选）
    world.debug.draw_box(box=carla.BoundingBox(transform.location, extent * 1e-2), rotation=transform.rotation, life_time=100, thickness=0.5, color=carla.Color(r=255,g=0,b=0))

if __name__ == '__main__':
    main()
```
