# Syntex
Use JavaCC.

## Definition

TODO:
function
vars

### `stmts` 

```
stmts(): {}  
{
	(stmt())* 
}

stmt(): {} 
{
	( ";"
	| LOOKAHEAD(2) labeled_stmt() | expr() ";"
	| block()
	| if_stmt()
	| while_stmt()
	| dowhile_stmt()
	| for_stmt()
	| switch_stmt()
	| break_stmt()
	| continue_stmt()
	| goto_stmt()
	| return_stmt()
	)
}
```

### `if_stmt` 
如下，如果 `if()` 或 `else` 后只有一条语句，可以省略大括号。注意嵌套。

```
if_stmt(): {}
{
   <IF> "(" expr() ")" stmt() [LOOKAHEAD(1) <ELSE> stmt()]
}
```

### `while_stmt`

```
while_stmt(): {}
{
  <WHILE> "(" expr() ")" stmt()
}
```

### `for_stmt`

```
for_stmt(): {}
{
   <FOR> "(" [expr()] ";" [expr()] ";" [expr()] ")" stmt()
}
```

### 跳转语句

```
break_stmt(): {}
{
	<BREAK> ";" 
}

return_stmt(): {}
{
	LOOKAHEAD(2) <RETURN> ";"   // 函数没有返回值的情况
	| <RETURN> expr() ";"       // 函数有返回值的情况	 
}
```

## Expression
与 `if` 语句和 `while` 语句不同，表达式的结构有层次，原因是其中所使用的运算符有优先级。

```
expr(): {} 
{
       LOOKAHEAD(term() "=")
       term() "=" expr()    // 普通的赋值表达式
      | LOOKAHEAD(term() opassign_op())
       term() opassign_op() expr()    // 自我赋值的表达式
      | expr10()    // 比赋值表达式优先级更高的表达式
}
```

### opassign_op

```
opassign_op(): {}
{
	( "+="
	| "-="
	| "*="
	| "/="
	| "%="
	| "&="
	| "|="
	| "^="
	| "<<="
	| ">>="
	)
}
```

### expr10
TODO: expr10

```
expr10(): {}
{
    expr9() ["?" expr() ":" expr10()]
}

expr9(): {} 
{
	expr8() ("||" expr8())*
}

expr8(): {} 
{
	expr7() ("&&" expr7())*
}

expr7(): {} 
{
    expr6() ( ">" expr6() 
            | "<" expr6() 
            | ">=" expr6() 
            | "<=" expr6() 
            | "==" expr6()
            | "!=" expr6() )*
}

expr6(): {} {
	expr5() ("|" expr5())*
}

expr5(): {} 
{
    expr4() ("^" expr4())*
}

expr4(): {} {
    expr3() ("&" expr3())*
}

expr3(): {} 
{
	expr2() ( ">>" expr2() 
	        | "<<" expr2()
	        )*
}

expr2(): {}
{
	expr1() ( "+" expr1()
	        | "-" expr1()
	        )*
}

expr1(): {} 
{
	term() ( "*" term() 
	       | "/" term() 
	       | "%" term()
	       )*
}
```

## `term` 项的分析

### 项的规则

```
term(): {} 
{
	// 带显示类型转换
	LOOKAHEAD("(" type()) "(" type() ")" term()
	| unary() 
}
```

### 前置运算符的规则
`term()` 是能添加 cast(类型转换) 运算符的 unary()。延用了 C 语言中的规范。

```
unary(): {}
{
	  "++" unary()    // 前置 ++
	| "--" unary()    // 前置 --
	| "+" term()      // 一元 +
	| "-" term()     
	| "!" term()      // 逻辑非
	| "~" term()      // 按位取反
	| "*" term()      // 指针引用
	| "&" term()      // 地址运算符
	| LOOKAHEAD(3) <SIZEOF> "(" type() ")"    //siziof(type) 某类型的大小
	| <SIZEOF> unary()
	| postfix()
}
```

### 后置运算符的规则

```
postfix(): {}
{
    primary()
    ( "++"            // 后置++
    | "--"            // 后置--
    | "[" expr() "]"  // 数组引用
    | "." name()      // 结构体或联合体的成员的引用
    | "->" name()     // 通过指针的结构体或联合体的成员的引用
    | "(" args() ")"  // 函数调用
    )*
}
args(): {} 
{
	[ expr() ("," expr())* ]
}
```

### 字面量的规则

```
primary(): {}
{
	  <INTEGER>
	| <CHARACTER>
	| <STRING>
	| <IDENTIFIER>
	| "(" expr() ")"
}
```
