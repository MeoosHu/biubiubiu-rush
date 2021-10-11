### 代码规范

**知行合一**

- 应当了解相关原则、模式和实践的理论知识，穷尽应知之事，并通过不断实践掌握

- 阅读大量代码，琢磨某段代码好在什么地方，坏在什么地方

**稍后等于永不（Later equals never）**

- 请花时间让自己做的更快，消除重复和提高表达力是值得持续努力的一件事，毕竟读与写的时间比超过了10:1
- 童子军军规：让营地比你来时更干净

#### 1.命名

**核心理念：表达力准确**，下述是命名的一些简单规则

#####  **1.1 名副其实**

例如下列代码：

```
public List<int[]> getThem(){
	List<int[]> list1 = new ArrayList<int[]>();
	for(int[] x : theList)
		if(x[0]==4)
			list1,add(x);
	return list1;	
}
```

也许你会冒出来以下疑问：

1. theList是什么类型的东西？
2. theList零下标条目的意义是什么？
3. 值4的意义是什么？
4. 我怎么使用返回的列表？

对它加以重构，就会变得耳目一新

```
public List<Cell> getFlaggedCells(){
	List<Cell> flaggedCells = new ArrayList<Cell>();
	for(Cell cell:gameBoard)
		if(cell.isFlagged)
			flaggedCells.add(cell);
	return flaggedCells;
}
```

**最佳实践**

1. 一旦发现有好的名称，立刻换掉
2. 函数、变量、类的名称应当可以告诉你：存在的目的，能做什么事，应该怎么用

##### **1.2 避免误导**

- 尽量少用accountList指代一组账号，除非它一直真的是List类型，否则容易引起**误导**，应采用bunchOfAccounts或accountGroup，甚至accounts
- 提防使用不同之处较小的名称
- 拼写前后不一致就是误导
- 禁用小写字母I和大写字母O作为变量名

##### **1.3 一个词应只有一个意义**

- **做有意义的区分**，如Variable不应出现在变量名中，table不应出现在表名中
- **避免思维映射**，不应让读者在脑海中把你的名称翻译为他们所熟知的名称
- **每个抽象概念选一个词**，例如get、retrieve、fetch都表示获取，此时应该统一概念，否则阅读代码的人可能会以为有多种含义
- **别用双关语**，避免将同一单词用于不同目的，不同概念，如add用于增加新纪录，此时需要有一个方法将单个参数放入集群，就不应命名为add

##### **1.4 让你的名称专业化**

- **使用解决方案域名称**，取个技术性的名称，如计算机科学、术语、算法名、模式名、数学术语
- **使用源自所涉问题领域的名称**，此时就知道去请教所在领域的专家，而不是自己瞎琢磨

##### **1.5 读的次数比写多**

- **使用读的出来的名称**
- **使用可搜索的名称**
- **避免使用编码**，例如用m_来表示成员变量，抽象工厂用XxxFactory即可，也没必要用IXxxFactory来表明它是个接口

##### **1.6 类名和方法名**

- 类名和对象名应该是名词或名词短语，避免使用Manager、Processor、Data、Info
- 方法名应该是动词或动词短语，属性访问器、修改器、断言应该加上set、get、is等前缀，重载构造器时，使用描述了参数的静态工厂方法名

##### **1.7 语境**

- **添加有意义的语境**，例如将变量放置一个类中，将算法分解为更小的函数（利用函数名添加语义）
- **不要添加没意义的语境**，短名称比长名称好，命名应该以精确为准

#### 2.函数

##### **2.1 短小**

- 20行封顶，每个函数都只说一件事
- if、else、while其中的代码块应该只有一行，函数名能够提供具有说明性的名称
- 函数的缩进层级不该多于一层或两层

##### **2.2 只做一件事**

判断函数是否只做了一件事，就是看能否再拆出一个函数，该函数不应是重新诠释其实现判断函数是否只做了一件事，就是看能否再拆出一个函数，该函数不应是重新诠释其实现

##### **2.3 每个函数一个抽象层级**

自顶向下读代码：向下规则（即每个函数后面都跟着位于下一抽象层级的函数，阅读代码可以层层递进的阅读）

##### **2.4 switch语句**

switch语句天生要做n件事，但要确保每个switch语句都埋藏在较低的抽象层级，而且永远不重复

如下列案例：

```
public Money calculatePay(Employee e) throw InvalidEmployeeType{
	switch(e.type){
		case COMMISSIONED:
			return calculateCommissionedPay(e);
		case HOURLY:
			return calcalateHourlyPay(e);
		case SALARIED:
			return calculateSalariedPay(e):
		default:
			throw new InvalidEmployeeType(e.type);
	}
}
```

这个函数有点长，增加新的类型，需要修改代码，违反了开闭原则；其次它远远不止做了一件事；而且违反了单一职责原则，可对它进行重构

```
public abstract class Employee{
	public abstract boolean isPayDay();
	public abstract Money calaulatePay();
	public abstract void deliverPay();
}

public interface EmployeeFactory{
	public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType;
}

public class EmployeeFactoryImpl implements EmployeeFactory{
	public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType{
		switch(r.type){
            case COMMISSIONED:
                return new CommissionedEmployee(r);
            case HOURLY:
                return new HourlyEmployee(r);
            case SALARIED:
                return new SalariedEmployee(r):
            default:
                throw new InvalidEmployeeType(e.type);
		}
	}
}
```

将switch语句埋到抽象工厂底下，同时使用switch为Employee的派生类创建合适的实体

##### **2.5 使用描述性的名称**

- 别害怕长名称，长而具有描述性的名称，比短而费解的名称好得多
- 命名方式应保持一致，使用与模块名一脉相承的短语、名词和动词

##### **2.6 函数参数**

应尽量避免使用三元参数，函数参数越少越好

- 一元函数

  - **操作参数或转换**，有输入有输出，应选用能区别这两种的名称
  - **事件**，有输入无输出，选用名称和上下文语境应让读者理解它是个事件

- 标识参数

  不要传入布尔值参数，往往意味着一个函数要做两件事，更好的做法是拆分为两个函数

- 二元函数

  应利用一些机制将其转换为一元函数

- 参数对象

  参数有两个、三个或三个以上考虑封装为对象

- 参数列表

  有可变参数的函数有可能是一元、二元...  **无法清晰明了**，建议少使用

- 动词和关键字

  对于一元函数，函数和参数应当形成一种非常良好的动词/名词对形式，如write(name)

##### **2.7 函数应无副作用**

- 时序性耦合

  如checkPassword函数中包含了初始化session操作，这就是副作用，应重命名函数为checkPasswordAndInitializeSession，并且违背了单一职责原则

- 避免使用输出参数

  **若函数必须修改某种状态，就修改所属对象的状态**。

  输出参数可以用this替代，例如appendFooter(s)替换为report.appendFooter()

  原函数声明为  public void appendFooter(StringBuffer report)

##### **2.8 分隔指令与询问**

函数要么修改某对象的值，要么返回该对象的有关信息，不可关联在一起，会造成语义混淆

##### **2.9 使用异常替代返回错误码**

- 不应在if中将指令当作表达式使用

  会导致深层次的嵌套结构，应该使用异常替代else部分将错误处理代码抽离出来

- 抽离try/catch代码块

  将try和catch代码块的主题部分抽离出来，单独形成函数

- 错误处理就是一件事（单一职责）

- Error.java依赖磁铁

  即某个类或枚举定义了所有错误码，每次修改都会导致依赖该类的类重新编译和部署，应使用异常替代错误码，新异常就可以从异常类派生出来，无需重新编译和部署

##### **2.10 结构化编程**

定义：每个函数都应该有一个入口、一个出口，即没有break、continue、goto等。

函数只要保持短小，出现break、continue无伤大雅，但应避免goto的使用

