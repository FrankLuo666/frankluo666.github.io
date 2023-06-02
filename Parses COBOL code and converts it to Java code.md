---
title: "解析COBOL代码并将其转换为Java代码"
date: 2023-06-01T10:58:09+09:00
draft: false
categories:
- IT
tags:
- cobol
- java
---



### 1、解析MOVE TO语句

#### ① pattern1，move to 语句在同一行:

比如，要将以下Cobol代码：

```cobol
333000         MOVE  exist_error  TO check_result
333100         MOVE  flag_ON        TO loop_flag
```

转为以下java代码：

```java
check_result = exist_error;
loop_flag = flag_ON;
```

可以用如下java代码进行进行转换：

```java
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class CobolParser {
    public static void main(String[] args) {
        String cobolCode = "333000         MOVE  exist_error  TO check_result\n"
                         + "333100         MOVE  flag_ON        TO loop_flag";

        // 使用正则表达式匹配COBOL代码中的MOVE TO语句
        Pattern pattern = Pattern.compile("MOVE\\s+(\\S+)\\s+TO\\s+(\\S+)");
        Matcher matcher = pattern.matcher(cobolCode);

        // 遍历匹配结果并生成对应的Java代码
        while (matcher.find()) {
            String sourceVariable = matcher.group(1);
            String targetVariable = matcher.group(2);

            String javaCode = targetVariable + " = " + sourceVariable + ";";
            System.out.println(javaCode);
        }
    }
}
```



- `\\s+` 表示匹配一个或多个空白字符（包括空格、制表符等）。
- `(\\S+)` 是一个捕获组，用于匹配一个或多个非空白字符。这里的 `( )` 表示捕获组，`\\S+` 表示匹配一个或多个非空白字符。因为要在Java字符串中表示一个反斜杠，所以需要使用双反斜杠进行转义。

当你使用`matcher.find()`进行迭代匹配时，以下方法会变得有意义：

- `matcher.group(1)` 返回与第一个捕获组（即 `(\\S+)`）匹配的字符串，即MOVE语句的源变量。
- `matcher.group(2)` 返回与第二个捕获组（即 `(\\S+)`）匹配的字符串，即MOVE语句的目标变量。

<br>

---

如果在此基础上，还**需要将每个变量的定义也打印出来**， 比如：

```java
# 在java中定义变量：
String exist_error;
String flag_ON;
```

可以将变量保存在List中再遍历出来， 修改后的代码如下：

```java
import java.util.Set;
import java.util.LinkedHashSet;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class CobolParser {
    public static void main(String[] args) {
        String cobolCode = "333000         MOVE  exist_error  TO check_result\n"
                         + "333100         MOVE  flag_ON        TO loop_flag";

        // 使用正则表达式匹配COBOL代码中的MOVE TO语句
        Pattern pattern = Pattern.compile("MOVE\\s+(\\S+)\\s+TO\\s+(\\S+)");
        Matcher matcher = pattern.matcher(cobolCode);
				
      	LinkedHashSet<String> variableSet = new LinkedHashSet<>();
      
        // 遍历匹配结果并生成对应的Java代码
        while (matcher.find()) {
            String sourceVariable = matcher.group(1);
            String targetVariable = matcher.group(2);
						
          	// 将变量加入list
          	// 如有全局变量不需要定义的，在这儿加一个判断：!sourceVariable.startsWith()
          	variableSet.add(sourceVariable);
            variableSet.add(targetVariable);
          
            String javaCode = targetVariable + " = " + sourceVariable + ";";
            System.out.println(javaCode);
        }
      
      	System.out.println("// 定义变量：");
      	for (String variable : variableSet) {
          	// 添加注释
          	String comment = "//" + variable;
            System.out.println(comment);
          	// 定义变量
          	String variableDefinition = "String " + variable + ";";
          	System.out.println(variableDefinition);
        }
    }
}
```



#### ② pattern2，move to 语句不在同一行:

如果COBOL代码中的MOVE和TO不在同一行，需要修改代码来处理这种情况。逐行读取COBOL代码，然后在每行中查找MOVE和TO语句：

```java
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class CobolParser {
    public static void main(String[] args) {
        String cobolCode = "333000         MOVE  exist_error\n"
                         + "333100         TO flag_ON\n"
                         + "333200         MOVE  exist_flag\n"
                         + "333000         TO flag_OFF";

        // 使用正则表达式匹配COBOL代码中的MOVE和TO语句
        Pattern movePattern = Pattern.compile("MOVE\\s+(\\S+)");
        Pattern toPattern = Pattern.compile("TO\\s+(\\S+)");

        String sourceVariable = null;
        String targetVariable = null;

        // 遍历COBOL代码的每一行
        String[] lines = cobolCode.split("\n");
        for (String line : lines) {
            Matcher moveMatcher = movePattern.matcher(line);
            Matcher toMatcher = toPattern.matcher(line);

            // 如果找到MOVE语句，则获取源变量
            if (moveMatcher.find()) {
                sourceVariable = moveMatcher.group(1);
            }

            // 如果找到TO语句，则获取目标变量并生成Java代码
            if (toMatcher.find()) {
                targetVariable = toMatcher.group(1);
                if (sourceVariable != null && targetVariable != null) {
                    String javaCode = targetVariable + " = " + sourceVariable + ";";
                    System.out.println(javaCode);
                    // 重置变量以处理下一组MOVE TO语句
                    sourceVariable = null;
                    targetVariable = null;
                }
            }
        }
    }
}
```

运行上面的代码，会将`cobolCode`	变量中的cobol代码转为java代码，结果如下：

```java
flag_ON = exist_error;
flag_OFF = exist_flag;
```

<br>

---

如果在此基础上，还需要将每个变量的定义也打印出来， 比如：

```java
# 在java中定义变量：
String exist_error;
String flag_ON;
```

可以将变量保存在List中再遍历出来， 修改后的代码如下：

```java
import java.util.List;
import java.util.ArrayList;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class CobolParser {
    public static void main(String[] args) {
        String cobolCode = "113000         MOVE  C_銘柄マスタ存在エラー\n"
                         + "113100         TO W_チェック結果\n"
                         + "113200         MOVE  C_ON\n"
                         + "113300         TO W_ループフラグ";

        // 使用正则表达式匹配COBOL代码中的MOVE和TO语句
        Pattern movePattern = Pattern.compile("MOVE\\s+(\\S+)");
        Pattern toPattern = Pattern.compile("TO\\s+(\\S+)");

        String sourceVariable = null;
        String targetVariable = null;

      	List<String> variableList = new ArrayList<>();

        // 遍历COBOL代码的每一行
        String[] lines = cobolCode.split("\n");
        for (String line : lines) {
            Matcher moveMatcher = movePattern.matcher(line);
            Matcher toMatcher = toPattern.matcher(line);

            // 如果找到MOVE语句，则获取源变量并添加到集合
            if (moveMatcher.find()) {
                sourceVariable = moveMatcher.group(1);
                // 如有全局变量不需要定义的，在这儿加一个判断：!sourceVariable.startsWith()
                variableList.add(sourceVariable);
            }

            // 如果找到TO语句，则获取目标变量并生成Java代码
            if (toMatcher.find()) {
                targetVariable = toMatcher.group(1);
                if (sourceVariable != null && targetVariable != null) {
                    // 检查目标变量是否已经定义，如果未定义则生成变量定义代码
                    if (!variableList.contains(targetVariable)) {
                        variableList.add(targetVariable);
                    }

                    String javaCode = targetVariable + " = " + sourceVariable + ";";
                    System.out.println(javaCode);
                    // 重置变量以处理下一组MOVE TO语句
                    sourceVariable = null;
                    targetVariable = null;
                }
            }
        }
      
      	System.out.println("// 定义变量：");
      	for (String variable : variableList) {
          	// 添加注释
          	String comment = "//" + variable;
            System.out.println(comment);
          	// 定义变量
          	String variableDefinition = "String " + variable + ";";
          	System.out.println(variableDefinition);
        }
    }
}
```



---

如果还需要将`MOVE TO`中的`OF`后面的对象也打印出来，例如：

如果cobol代码是这样的：

```cobol
333000         MOVE  exist_error OF before
333100         TO exist_error OF after
```

需要转为伪java代码如下： 

```java
after.exist_error = before.exist_error; 
```

那么解析cobol的java代码就要这么写：

```java
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class CobolToJavaConverter {
    public static void main(String[] args) {
        String cobolCode = "333000         MOVE  exist_error OF before\n"
                         + "333100         TO exist_error OF after";
        String javaCode = convertCobolToJava(cobolCode);
        System.out.println(javaCode);
    }

    public static String convertCobolToJava(String cobolCode) {
        StringBuilder javaCodeBuilder = new StringBuilder();
        String[] lines = cobolCode.split("\n");

        Pattern pattern = Pattern.compile("MOVE\\s+(\\S+)\\s+OF\\s+(\\S+)\\s+TO\\s+(\\S+)\\s+OF\\s+(\\S+)");
        for (String line : lines) {
            Matcher matcher = pattern.matcher(line);
            if (matcher.find()) {
                String sourceField = matcher.group(1);
                String sourceObject = matcher.group(2);
                String targetField = matcher.group(3);
                String targetObject = matcher.group(4);

                String javaStatement = targetObject + "." + targetField + " = " + sourceObject + "." + sourceField + ";";
                javaCodeBuilder.append(javaStatement).append("\n");
            }
        }

        return javaCodeBuilder.toString();
    }
}

```



### 2、解析赋值语句

例如，将cobol代码的赋值语句：

```cobol
03  C_end         PIC  X(01) VALUE '1'.
03  H_check       PIC  X(02) VALUE '00'.
03  H_number      PIC  9(08).
```

转为java代码：

```java
private final String C_end = "1";
String H_check = "00";
String H_number = 0;
```

可以用如下的java代码进行解析：

```java
import java.util.HashMap;
import java.util.Map;

public class CobolParser {
    private Map<String, String> variableMap;

    public CobolParser() {
        variableMap = new HashMap<>();
    }

    public void parseCobolCode(String cobolCode) {
        String[] lines = cobolCode.split("\n");
        for (String line : lines) {
            if (line.contains("PIC")) {
                String variableName = line.substring(0, line.indexOf("PIC")).trim();
                String picStatement = line.substring(line.indexOf("PIC") + 3).trim();
                String javaType = convertPicToJavaType(picStatement);
                String javaCode = generateJavaCode(variableName, javaType);

                variableMap.put(variableName, javaCode);
            }
        }
    }

    private String convertPicToJavaType(String picStatement) {
        if (picStatement.startsWith("X")) {
            return "String";
        } else if (picStatement.startsWith("9")) {
            return "int";
        }

        // Add support for other PIC types if needed

        return "";
    }

    private String generateJavaCode(String variableName, String javaType) {
        StringBuilder sb = new StringBuilder();
        sb.append("// ").append(variableName).append("\n");
        sb.append("private final ").append(javaType).append(" ").append(variableName).append(" = ");

        if (javaType.equals("String")) {
            sb.append("\"\"");
        } else if (javaType.equals("int")) {
            sb.append("0");
        }

        sb.append(";\n");

        return sb.toString();
    }

    public Map<String, String> getVariableMap() {
        return variableMap;
    }

    public static void main(String[] args) {
        String cobolCode = "03  C_end         PIC  X(01) VALUE '1'.\n" +
                "03  H_check       PIC  X(02) VALUE '00'.\n" +
                "03  H_number      PIC  9(08).\n";

        CobolParser parser = new CobolParser();
        parser.parseCobolCode(cobolCode);

        Map<String, String> variableMap = parser.getVariableMap();
        for (String variableName : variableMap.keySet()) {
            System.out.println(variableMap.get(variableName));
        }
    }
}
```

