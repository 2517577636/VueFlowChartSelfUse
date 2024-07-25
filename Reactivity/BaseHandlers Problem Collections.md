# BaseHandlers Problem Collections

### 1. 在获取属性ReactiveFlags.RAW值为啥需要通过比较当前receiver 是否存在于记录对象中？而不是直接返回target对象？为啥需要额外判断原型对象是否相等？

```
- 前提条件：实现toRaw函数（直接获取目标值）。
  1. proxy 与 普通对象（Object）无法进行有效区分。
 【Ps: 无法使用Object.prototype.toString.call(proxy) 来获取proxy类型】

  2. 当toRaw函数参数target的prototype是Reactive对象时，根据函数功能需要直接返回target
  而不是返回原型Reactive对应的属性值【v_raw】。

  3. 当toRaw函数参数target是Reactive的proxy时，需要返回Reactive的原始目标对象。
  【Ps: 因此toRaw内部需要递归】。
```
**解答**：
1. 直接返回target时，当toRaw函数参数为上述第二点时，toRaw无法判断属性是target对象上，还是其所属原型对象上。所以，无法直接返回target。那么这里为啥还需要比较当前receiver 是否存在于记录对象中，直接判断当前target对象是否在记录对象中存在值，不就可以吗？（Ps: 即为啥 receiver === reactiveMap.get(target) 需要这么写？而不是 reactiveMap.get(target)）
- 根据proxy特性，target永远为最初的target，receiver则为调用get方法时的对象。所以，这里reactiveMap.get(target) 是可以得到对应值。从而导致预期值不为false。

1. 当toRaw函数参数target是Reactive的proxy时，需要返回Reactive的原始目标对象。为满足这个需求，需要使用原型链进行对比。（Ps: Proxy本身没有prototype属性，当创建实例对象时，它的prototype被赋值为target的prototype。）

### 2. 
