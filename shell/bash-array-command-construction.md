---
GeneratedBy: ChatGPT (Bash Array Command Builder Summary)
Date: 2025-11-28
---

# Bash：使用数组构建多级命令与参数拼装的最佳实践

> **适用于：sshpass + rsync / scp / ssh、sudo 前缀、env 前缀、复杂命令构建、动态参数拼接。**  
> 这是编写安全、健壮、可维护 Bash 脚本的核心技巧。

## 目录

1. 为什么不要在变量里存引号语法  
2. 为什么构建命令必须用数组而不是字符串  
3. 正确展开数组：${array[@]} 与 "${array[@]}" 的区别  
4. 构建分级命令：prefix 数组（如 sshpass）  
5. 示例：sshpass + rsync/scp/ssh 的数组构建方法  
6. 数组方式 vs eval 方式  
7. 总结：编写安全 Bash 的黄金规则

---

## 为什么不要在变量里存引号语法

错误方式（常见坑）：

```
opt="--include='*.sh'"
cmd="rg $opt pattern"
eval "$cmd"
```

问题：

- 变量里包含 shell 引号语法不可能安全  
- 展开不会重新解析引号，导致必须 eval  
- eval 会导致注入风险、难维护  

**原则：变量只存“数据”，引号语法由代码负责。**

---

## 为什么构建命令必须用数组而不是字符串

字符串方式很容易引发：

- word splitting（空格导致参数被拆）  
- glob expansion（*、? 等导致文件名展开）  
- quote 嵌套陷阱  
- 最终一般要靠 eval 才能执行  

数组方式：

```
cmd=(grep "pattern with space" "file name.txt")
"${cmd[@]}"
```

优点：

- 每个数组元素都是一个“参数原子”  
- 不会 split  
- 不会 glob  
- 不需要 eval  
- 引号不需要你手工管理  

---

## 正确展开数组

必须记住的一句话：

**始终引用数组：`"${array[@]}"`  
不引用 `${array[@]}` → 会触发 word splitting → 参数数量改变。**

示例：

```
arr=("a b" "c d")

"${arr[@]}"   → 2 个参数：["a b", "c d"]  
${arr[@]}     → 4 个参数：["a", "b", "c", "d"]
```

---

## 构建分级命令：prefix 数组

当命令有前缀工具（如 sshpass、sudo、env）时：

坏例（字符串方式）：

```
prefix="sshpass -p $password"
cmd="$prefix rsync -avz ..."
eval "$cmd"
```

必须 eval → 极不安全。

### 好例：prefix 也用数组

```
password="abc def"
prefix=(sshpass -p "$password")
```

prefix 数组元素：

```
prefix[0] = sshpass  
prefix[1] = -p  
prefix[2] = abc def
```

组合命令：

```
cmd=("${prefix[@]}" rsync -avz "$src" "$dest")
"${cmd[@]}"
```

---

## sshpass + rsync/scp/ssh 数组示例

```
password="p@ss' with space"
src="/local path/with space/"
dest="user@example.com:/remote path/with space/"

prefix=(sshpass -p "$password")

cmd=("${prefix[@]}" rsync -avz -e ssh "$src" "$dest")
"${cmd[@]}"

cmd=("${prefix[@]}" scp "$src" "$dest")
"${cmd[@]}"

cmd=("${prefix[@]}" ssh "user@example.com" "hostname")
"${cmd[@]}"
```

---

## 数组方式 vs eval 方式

| 特性 | 数组方式 | eval 方式 |
|------|----------|------------|
| 安全性 | ⭐⭐⭐⭐⭐ | ⭐ |
| 是否需要管理引号 | ❌ | 必须手动管理 |
| 空格安全 | ✔ | ❌ |
| glob 安全 | ✔ | ❌ |
| 注入风险 | 极低 | 高 |
| 可维护性 | 高 | 低 |
| 是否重新解析变量内容 | 不会 | 会（危险） |

---

## 总结：黄金规则

### 1. 变量存“数据”，不存 shell 语法  
### 2. 用数组构建命令与参数，避免字符串拼接  
### 3. 永远使用 `"${array[@]}"` 来展开数组  
### 4. prefix（sshpass、sudo 等）也应该是数组  
### 5. eval 是最后手段，如非必要绝不用

通过数组构建命令，是 Bash 中唯一安全、可维护、避免引号和 split 陷阱的方式。
