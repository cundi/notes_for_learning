# Inspect moudle

inspect.signature（fn)将返回一个inspect.Signature类型的对象，值为fn这个函数的所有参数

inspect.Signature对象的paramerters属性是一个mappingproxy（映射）类型的对象，值为一个有序字典（Orderdict)。

这个字典里的key是即为参数名，str类型

这个字典里的value是一个inspect.Parameter类型的对象，根据我的理解，这个对象里包含的一个参数的各种信息

inspect.Parameter对象的kind属性是一个_ParameterKind枚举类型的对象，值为这个参数的类型（可变参数，关键词参数，etc）

inspect.Parameter对象的default属性：如果这个参数有默认值，即返回这个默认值，如果没有，返回一个inspect._empty类。