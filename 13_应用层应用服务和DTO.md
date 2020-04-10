# 13 应用层应用服务和DTO

## 13.1 任务描述

### 13.1.1 任务介绍

为领域层添加实体类。

### 13.1.2 任务要求

- 了解领域驱动设计中的领域层
- 实体类的实现
  - 使用继承实现通用的属性
  - 使用默认构造函数实现相关默认值的初始化
  - 通过注解定义Annotation，判断属性的值是否合法

## 13.2 工作指导说明

应用服务用于将领域(业务)逻辑暴露给展现层。展现层通过传入DTO(数据传输对象)参数来调用应用服务，而应用服务通过领域对象来执行相应的业务逻辑并且将DTO返回给展现层。

```c#
public class UserAppService : IUserAppService
{
    private readonly IRepository<User> _userRepository;

    public UserAppService(IRepository<User> userRepository)
    {
        _userRepository = userRepository;
    }

    public void CreateUser(CreateUserInput input)
    {
        var user = _userRepository.FirstOrDefault(p => p.Phone == input.Phone);
        if (user != null)
        {
            // 抛出用户友好异常
        }

        user = new User { Name = input.Name, Phone = input.Phone }; // 根据实际情况进行初始化
        _UserRepository.Insert(User);
    }
}
```

### 13.2.2 DTO

数据传输对象（DTO)(Data Transfer Object)，是一种设计模式之间传输数据的软件应用系统。数据传输目标往往是数据访问对象从数据库中检索数据。数据传输对象与数据交互对象或数据访问对象之间的差异是一个以不具有任何行为除了存储和检索的数据（访问和存取器）。

数据传输对象(Data Transfer Objects)用于应用层和展现层的数据传输。  
展现层传入数据传输对象(DTO)调用一个应用服务方法，接着应用服务通过领域对象执行一些特定的业务逻辑并且返回DTO给展现层。这样展现层和领域层被完全分离开了。在具有良好分层的应用程序中，展现层不会直接使用领域对象(仓库，实体)。  

为什么要用DTO，为什么不直接使用实体？

1. 层的隔离
2. 数据隐藏
3. 序列化的需要

命名规范：CreateUserInput，SearchUserInput，SearchUserOutput，UserDto等。

### 13.2.2.1 DTO的验证

### 13.2.2.2 与实体的自动映射

### 13.2.2.3 列表页中用到的DTO

## 13.3 产品要求

无

## 13.4 工作要求

无
