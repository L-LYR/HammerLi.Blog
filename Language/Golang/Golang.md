---
Author: HammerLi
Date: 2021-07-13
---

# Golang零散知识

---

>  Golang代码规范[2021-07-13]
>
>  学习单元测试的编写[2021-07-23]
>
>  Golang包管理[2021-08-02]

----

# 注释

- 因为Go语言采用包的方式来结构化管理代码，同一个功能模块的代码一般放在同一个包中，但同一功能模块的代码可能分散在很多的文件中，包名能够表示的信息是有限的，因此包注释是极其必要的，它需要包含一个包的大致内容、功能与使用方法。包注释以`Package xxx`开头，并放在包的一个文件的首部；包注释如果内容过多，可以放在`doc.go`文件中。

- Go语言采用首字母大小写来控制一个包内名称的外部可见性。因此，对于外部可见的名称是有必要使用一段注释来解释其作用。该注释以名称开头，自定义类型名称需要加上冠词。

- 对于代码块中的注释，单独一行，一个完整的句子；或者代码末尾，一个完整的短语。

# 命名

- 通用
  - 使用驼峰命名法。

- 包名
  - 仅小写字母组成，使用单数名词。
  - 简短且包含一定上下文信息。
  - 如果包名的前缀能作为路径则可以将不同功能拆分为子包。
  - 不使用常用变量名称作为包名。
  - 在保证可读性的前提下，谨慎使用缩写。
- 函数名
  - 不携带包名信息。
  - 在不与上下文信息重复的条件下，可以包含返回类型名。
  - `New`函数可以作为包内全部方法入口实例的构造函数。
  - 结构体的未导出成员的`getter`应命名为该成员的首字母大写名称。
  - 结构体的方法不要使用OOP中的常用名，一般使用一两个字母来表示之前的类型，且必须统一。
- 接口名
  - 对于只有一个方法的接口，通常将其命名为方法名称之后加上`-er`。
- 变量名
  - 缩略词全大写，但如果在开头且不导出则全小写。

# import分组

- 使用`goimports`来进行分组管理，标准库为第一组，组内字典序排列。
- 如果仅需要调用包内`init()`方法，则只需要将包重命名为`_`即可。
- 仅当“测试代码依赖的库“依赖”需要被测试的库”的时候，可以使用`.`作为别名来导入它，使其伪装成“需要被测试的库”的一部分。
- 仅当 import 的库有命名冲突或者库的名字与 import path 最后一个元素不匹配时，你可以指定一个别名来使用它。

# `panic`与`recover`

- 若问题可以被屏蔽或解决，建议使用`error`代替`panic`。
- 特殊地，当程序启动阶段发生不可逆转的错误时，可以在`init`或`main`函数中使用`panic`。
- `recover`只能在被`defer`的函数中使用（嵌套无法生效），并且只在当前`goroutine`生效，如果需要更多的上下文信息，可以`recover`后在 log 中记录当前的调用栈。

# 错误

- 简单的错误指的是仅出现一次的错误，且在其他地方不需要捕捉该错误，优先使用`errors.New`来创建匿名变量来直接表示该错误。如果有 format 的需求，使用`fmt.Errorf`。
- 一个普遍出现的错误应放入相应代码块的开头，多个则应放到文件开头并与普通变量区分开，特别多则应分类放置不同代码块开头。
- 错误内容尽量不要含有大写字母与标点符号，可以考虑在错误内容的开头添加包名来标注定位。
- 特别复杂且需要包含上下文信息的错误则需要定义在包内的`error.go`文件内。
- 使用Go1.13中新增的`errors`包内的方法`errors.Is `  `errors.As` `errors.Unwrap`。
- 参考
  - https://www.flysnow.org/2019/09/06/go1.13-error-wrapping.html
  - https://blog.golang.org/go1.13-errors

# 单元测试

单元测试一般用到一下几个库

```shell
testify   // 断言库
httptest  // 官方自带http请求模拟

// 各有优劣的打桩库
gomonkey
gomock
gostub
```

- 表驱动模式

  ```golang
  // twice.go
  func twice(i interface{}) (int, error) {
  	switch v := i.(type) {
  	case int:
  		return v * 2, nil
  	case string:
  		val, err := strconv.Atoi(v)
  		if err != nil {
  			return 0, fmt.Errorf("invalid string num %w", err)
  		}
  		return val * 2, nil
  	default:
  		return 0, errors.New("unkown type")
  	}
  }
  // twice_test.go
  func TestTwice(t *testing.T) {
  	type args struct {
  		i interface{}
  	}
  	cases := []struct {
  		name    string
  		args    args
  		want    int
  		wantErr bool
  	}{
  		{
  			name: "int",
  			args: args{i: 10},
  			want: 20,
  		},
  		{
  			name: "string success",
  			args: args{i: "11"},
  			want: 22,
  		},
  		{
  			name:    "string failed",
  			args:    args{i: "nop"},
  			wantErr: true,
  		},
  		{
  			name:    "unknown type",
  			args:    args{i: []byte("11")},
  			wantErr: true,
  		},
  	}
  
  	for _, c := range cases {
  		t.Run(c.name, func(t *testing.T) {
  			get, err := twice(c.args.i)
  			assert.Equal(t, c.wantErr, err != nil)
  			assert.Equal(t, c.want, get)
  		})
  	}
  }
  ```













