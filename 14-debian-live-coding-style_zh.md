# 编码风格

## 15.1 兼容性

* 避免使用"bashisms"，代码库必须符合POSIX标准，从而具有普遍的兼容性。

* 此外，它必须符合当前Debian政策选择的POSIX规范版本。

* 您可以使用'sh -n'和'checkbashisms'检查您的脚本。

* 确保所有shell代码都在'set -e'下运行。

## 15.2 缩进

* 始终使用制表符而不是空格。

* 保持case分支终止符（";;"）与分支的"内容"对齐，而不是分支"入口"。

  好的：

  ```
  case "${1}" in  
  foo)  
  foobar  
  ;;  
  
  bar)  
  foobar  
  ;;  
esac  
  ```

## 15.3 换行

* 通常，行的最大长度应为80个字符。

* 关键字如then和do的放置应根据混乱和可读性进行合理选择。特别是对于小段代码，应该优先将它们放在与它们相关的前一个关键字（if; for;等）同一行上。只有在符合最大行长限制的情况下才放在下一行。一个总是应该放在下一行的情况是，当它们后面的内容被分成多行时，这样放在新行上可以在它们与后续代码主体之间创建清晰的分隔。例如：

  推荐：

  ```
  if foo; then  
  bar  
  fi  
  
  for FOO in $ITEMS; do  
  bar  
  done  
  
  if [ "${MY_LOCATION_VARIABLE}" = "something" ] && [ -e "${MY_OUTPUT_FILE}" ]  
  then  
  MY_OTHER_VARIABLE="$(some_bin ${FOOBAR} | awk -F_ '{ print $1 }')"  
  fi  
  
  if [ "${MY_FOO}" = "something" ] && [ -e "path/${FILE_1}" ] ||  
  [ "${MY_BAR}" = "something_else" ] && [ ${ALLOW} = "true" ]  
  then  
  foobar  
  fi  
  ```

  不太理想：

  ```
  if [ "${MY_LOCATION_VARIABLE}" = "something" ] && [ -e "${MY_OUTPUT_FILE}" ]; then  
  MY_OTHER_VARIABLE="$(some_bin ${FOOBAR} | awk -F_ '{ print $1 }')"  
  fi  
  ```

  糟糕：

  ```
  if [ "${MY_LOCATION_VARIABLE}" = "something" ] && [ -e "${MY_OUTPUT_FILE}" ] || [ "${MY_LOCATION_VARIABLE}" = "something-else" ] && [ -e "${MY_OUTPUT_FILE_2}" ]; then  
  MY_OTHER_VARIABLE="$(some_bin ${FOOBAR} | awk -F_ '{ print $1 }')"  
  fi  
  ```

* 优先将函数的开括号放在新行上（以与已建立的风格保持一致），并保持括号与函数名对齐：

  好的：

  ```
  Foo ()  
  {  
  bar  
  }  
  ```

  不好的（与现有风格不一致）：

  ```
  Foo () {  
  bar  
  }  
  ```

  糟糕：

  ```
  Foo ()  
        {  
        bar  
        }  
  ```

## 15.4 变量

* 变量始终使用大写字母。

* 在_live-build_中使用的配置变量应以LB_前缀开头。

* 局部函数变量应限制在局部范围内。

* 与_live-config_中的引导参数相关的变量以LIVE_开头。

* _live-config_中的所有其他变量以_前缀开头。

* 使用大括号括住变量；例如，写\${FOO}而不是$FOO。

* 始终用引号保护变量以尊重潜在的空格（除非在必要时实现正确的单词拆分）：写"\${FOO}"而不是${FOO}。

* 出于一致性原因，始终在为变量赋值时使用引号：

  不好：

  ```
  FOO=bar  
  ```

  好：

  ```
  FOO="bar"  
  ```

* 如果使用多个变量，优先引用整个表达式：

  通常不好：

  ```
  if [ -f "${FOO}"/foo/"${BAR}"/bar ]; then  
  foobar  
  fi  
  ```

  好：

  ```
  if [ -f "${FOO}/foo/${BAR}/bar" ]; then  
  foobar  
  fi  
  ```

## 15.5 杂项

* 在调用sed时，优先使用"|"作为分隔符，例如"sed -e 's|foo|bar|'"（不带引号）。

* 不要使用test命令进行比较或测试，使用"["和"]"（不带引号）；例如，"if [ -x /bin/foo ]; ..."而不是"if test -x /bin/foo; ..."。

* 在代码比条件检查（if foo; ...和没有实际if关键字的测试，例如[ -e "${FILE}" ] || exit 0）更具可读性的地方使用case。

* 使用"Foo_bar"样式名称作为函数名，即首字母大写，然后全部小写，合理使用下划线以提高可读性。 