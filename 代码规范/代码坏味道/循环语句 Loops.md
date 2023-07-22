# 循环语句 Loop

Ref：《Effective Java 第三版》
Day：2021-12-12

## 重构方法

* [for-each 循环优于传统的 for 循环](../重构方法/for-each%20循环优于传统的%20for%20循环.md)
* [以管道取代循环](../重构方法/以管道取代循环.md)

## 示例代码
```javascript
function acquireData(input) {
    const lines = input.split(“\n”);
    let firstLine = true;
    const result = [];
    for (const line of lines) {
        if (firstLine) {
            firstLine = false;
            continue;
        }
        if (line.trim() === “”) continue;
        const record = line.split(“,”);
        if (record[1].trim() === “India”) {
            result.push({
                city: record[0].trim(),
                phone: record[2].trim()
            });
        }
    }
    return result;
}

// 重构后
function acquireData(input) {
    const lines = input.split("\n");
    return lines
        .slice(1)
        .filter(line => line.trim() !== "")
        .map(line => line.split(","))
        .filter(fields => fields[1].trim() === "India")
        .map(fields => ({
            city: fields[0].trim(),
            phone: fields[2].trim()
        }));
}

```

## 