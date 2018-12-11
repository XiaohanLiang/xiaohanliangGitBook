# 事例-1

```text
nmap <silent> ;s :call ToggleSyntax() <CR>
```

| **按键名称** | 作用 |
| :--- | :--- |
| namp | normal-mode key mapping |
| &lt;silent&gt; | 具体执行了什么内容不用说出来 |
| ;s | 出现这两个关键字的时候准备映射 |
| :call Toggle\( \) | 调用Toggle函数，无参 |
| &lt;CR&gt; | 执行一下[回车](https://xiaohanliang.gitbook.io/notes/v/vimscript/syntax/te-shu-an-jian-ying-she) |

