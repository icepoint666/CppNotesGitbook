# mutable

`mutable` 是用来修饰一个 `const` 示例的部分可变的数据成员的。如果要说得更清晰一点，就是说 `mutable` 的出现，将 C++ 中的 `const` 的概念分成了两种。

* 二进制层面的 `const`，也就是「绝对的」常量，在任何情况下都不可修改（除非用 `const_cast`）。
* 引入 `mutable` 之后，C++ 可以有逻辑层面的 `const`，也就是对一个常量实例来说，从外部观察，它是常量而不可修改；但是内部可以有非常量的状态。

显而易见，`mutable` 只能用来修饰类的数据成员；而被 `mutable` 修饰的数据成员，可以在 `const` 成员函数中修改。

```cpp
class HashTable {
 public:
    //...
    std::string lookup(const std::string& key) const
    {
        if (key == last_key_) {
            return last_value_;
        }

        std::string value{this->lookupInternal(key)};

        last_key_   = key;
        last_value_ = value;

        return value;
    }

 private:
    mutable std::string last_key_
    mutable std::string last_value_;
};
```

这里，我们呈现了一个哈希表的部分实现。**\(带缓存的哈希表需要用到mutable）**

显然，对哈希表的查询操作，在逻辑上不应该修改哈希表本身。因此，`HashTable::lookup` 是一个 `const` 成员函数

在 `HashTable::lookup` 中，我们使用了 `last_key_` 和 `last_value_` 实现了一个简单的「缓存」逻辑。当传入的 `key` 与前次查询的 `last_key_` 一致时，直接返回 `last_value_`；否则，则返回实际查询得到的 `value` 并更新 `last_key_` 和 `last_value_`。

在这里，`last_key_` 和 `last_value_` 是 `HashTable` 的数据成员。按照一般的理解，`const` 成员函数是不允许修改数据成员的。但是，另一方面，`last_key_` 和 `last_value_` 从逻辑上说，修改它们的值，外部是无有感知的；因此也就不会破坏逻辑上的 `const`。为了解决这一矛盾，我们用 `mutable` 来修饰 `last_key_` 和 `last_value_`，以便在 `lookup` 函数中更新缓存的键值。

