rust中，将错误处理分为两类：1)不可恢复错误 `panic!` 。2）可恢复错误 `enum Result`



# panic!

`panic!` 宏：会终止代码的执行



# Result

```rust
enum Result<T, E> {
  Ok(T),
  Err(E),
}
```

* `Result` 定义了两个变体：代码执行成功时，返回的应该是 `Ok(T)`, 如果执行失败时，返回的应该是 `Err(E)`. 我们可以通过判断返回的是啥 来判断代码是否执行成功。



对Result对象的处理

* 失败时触发 `panic`. : `.unwrap(), .expect()`
* 错误传播：`? 运算符` 。
* 最原始方法：使用 `match`



 