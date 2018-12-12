---
description: '我又检查了一遍我的makefile, 我不觉得我缺少什么分隔符'
---

# missing separator.  Stop.

### Makefile对于TAB的检查方式极其愚蠢

针对所有制定的编译规则&所有操作，全都是以TAB的形式导出来的

1. 四个空格 $$\neq$$ 一个TAB
2. 在Goland下甚至一个TAB都不等于一个TAB
3. 只有在Vim下一个TAB才是一个TAB

### Goland是怎么制作一个TAB的

我们使用指令: `cat -e -t -v GNUmakefile` ， 这个指令会告诉我们这个Makefile到底会做什么

```text
//------------------------------------------------------------------ Correct
build:$
^Igo install github.com/jdclouddevelopers/terraform-provider-jdcloud$
//------------------------------------------------------------------ Incorrect
heihei:$
    echo idiot$
```

可以发现，Goland制作一个TAB，方式就是四个空格，这本质上跟你自己打四个空格也没什么区别，~~反正都是错的~~。你通过前后移动光标就能发现，正确的TAB会一次前进四个位置，Goland自己制作的TAB一次前进一个位置。于是Make会很疑惑你为什么会在这里打四个空格，然后报错。

### Vim中正确的TAB

从上面的代码片段里可以发现，一个正确的TAB会以**`^I`**的形式出现

