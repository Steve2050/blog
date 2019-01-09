## aws、idea、nodejs等坑
- 注意，lodash的foreach内部一样可以用函数的形式来使用await
- dynamodb获取所有row count的时候，目前只有2种方案：用scan去统计、用describe table去获取所有count（6小时会自动刷新一次）
- 获取js的嵌套属性的时候，一定不要用model[nextedPath]来获取，这样是无法获取到嵌套属性的！！！同理delete也是，例如：delete[`${nest.nest}`]，这是不会删除嵌套属性的，特别注意
- dynamodb的filterExpression以及conditionExpression公用一个条件，参考：https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Expressions.ConditionExpressions.html
- 为了动态的实现表字段的增删，处理方法之一：全表遍历fix和检查所有对象即可。可以并发处理。
- 为了动态的实现表字段的增删，处理方法之二：直接使用bigObjs保存大的map对象，每次login的时候，检测，创建即可。==在其它代码的地方直接使用嵌套对象即可==。性能消耗降到最低。修复策略：（1）每次系统有更新的时候，清理掉sessionId即可。让玩家重新登入，即可实现动态修复（2）修改player表的bigObjs中的某一个属性，即可快速修复该属性。
- dynamodb存储大的对象的解决方案：dd的item限制大小是400kb，看这里：https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/CapacityUnitCalculations.html 超过这个大小，就需要压缩或者采取其它策略了：https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-use-s3-too.html
- dynamodb的错误处理，例如返回错误码等信息：https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Programming.Errors.html
- dynamodb在更新大量的属性的文本的时候，使用map精确更新最好，对于检查属性是否存在，初始化等操作，可以通过一个token来统一做为更新条件。一个token在取得对象的时候，可以保证更新对象的值是正确的，所以大多数时候应该够用了。todoo：发现有bug的场景
- 特别注意lodash的_.concact([],'')，这种的结果数组length不为0，结果是length为1，但是undefined的元素
- idea的⌘+Q会错误的直接强管ide，关闭这个功能可以如下操作：System Preferences-》keyboard-》shotcuts-》app shotcuts-》选择对应的的app为idea，在idea下新增一个新的快捷方式：【Quit IntelliJ IDEA】为其他值，即可覆盖默认的⌘+Q。可以参考：https://blog.csdn.net/hlllmr1314/article/details/80596932?utm_source=blogxgwz2
- dynamodb的更新，父路径是必须存在的，例如parent.child，必须先更新parent为空对象{}才行。
- dynamodb的分页，limit并不是实际的数据，实际只会按照大小1mb来进行获取，LastEvaluatedKey是唯一的判定标准。
参考：https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Query.html#Query.Pagination

- 关于循环引用，当前的引用：FooUtils-》WptUtils-》CommonUtils-》Constants。即circular-references的问题，目前最佳方式仍然是：完全消除所有的循环引用，采用组合的方式来组织代码结构，例如A作为最基础的类，B、C都会使用A，然后D会组合B、C的功能，但是C里面绝对不应该引用D。保证代码结构统一，在最开始就直接export；具体查看：https://softwareengineering.stackexchange.com/questions/11856/whats-wrong-with-circular-references

- lambda界面的测试貌似并不能模拟apiGateway那边的post等测试。apiGateway要deploy后才会暴露公网地址。否则只能在内部进行测试。具体查看：https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html
- 关于dynamodb中的对象保存，如果全部保存在一个大数组中，反而不好维护，如果用对象的key，那么则可以非常精确的使用条件更新，且不会影响其它key-value的值。实际中持续关注
- dynamodb是必须要初始化的，如果不初始化则会使用默认区域，为了本地测试方便，需要在lambda中配置环境变量：local=false（字符串）
- lambda注意配置默认的超时时间，不配置就是3秒，还有关注内存。
- dynamodb的删除、创建、查询表，可以在本地进行快速测试，在实际环境中，实际创建时间长的多，无法创建后马上使用，除非用循环判定表状态！
- 对于integer的加减，考虑使用lodash的toInteger来进行安全的加减法，避免未声明的类型
- js函数的值传递问题：Primitives are passed by value, Objects are passed by "copy of a reference"
- await的问题，idea会有提示
- 对有``符号的情形，谨慎使用格式化，对比的时候必须打开空格和空行对比
- 对js，绝对禁止使用对象属性的重构，例如obj.attr这种的重构，会导致未知异常
- 可以考虑采用let a = a || value 这种形式来初始化，比if更简洁
- pm2 的日志管理，自带了清理日志、rotate日志功能（需要安装pm2 install pm2-logrotate）,see：https://pm2.io/doc/en/runtime/guide/log-management/，
