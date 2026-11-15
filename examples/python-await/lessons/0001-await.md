# 第一课：取得协程的结果（人工构造）

目标：说明调用 async def 创建的协程对象与 await 得到的结果有何区别。
术语：async def 定义协程函数；调用产生协程对象；这里用 await 执行该协程并取得返回值。
单独调用不会立即运行函数体；不要由此推断每一次 await 都必然发生可观察的任务切换。
参考入口见[RESOURCES](../RESOURCES.md)。以下是讲解例子，不是独立能力证据。

~~~python
import asyncio

async def get_value():
    return 7

async def main():
    value = await get_value()
    print(value)

asyncio.run(main())
~~~

预期输出为 7；没有实际运行结果时标注“未运行”。

练习 Q1：执行下面完整程序会按什么顺序输出？先等待作答；需要时只提示“调用与等待发生在不同位置”。

~~~python
import asyncio

async def number():
    print("inside")
    return 3

async def main():
    job = number()
    print("before")
    print(await job)

asyncio.run(main())
~~~

本案例后续作答见[会话一](../learning-records/0001-first-session.md)，实际教学不应在提问后立即附解析。
速查：协程对象表示待执行的计算；本例通过 await 得到普通返回值。真实学习可再建自己的术语表。
