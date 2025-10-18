-
- v-model
	- `v-model` 是一个 **双向数据绑定指令**
	- **从数据到视图：**将变量的值设置到元素的 value 属性。
	- **从视图到数据：**监听输入框的 `input` 事件，一旦用户输入，立即更新变量的值。
	- 在模板中使用 `v-model="newTodo"` 相当于同时做了两件事：
		- **绑定值 (`:value`)**: `<input :value="newTodo.value">`
		- **监听事件 (`@input`)**: `<input @input="newTodo.value = $event.target.value">`
		- **效果：** 无论你是在 JavaScript 中修改 `newTodo.value`，还是用户在输入框中打字，两者总是保持同步。
- `v-for`
	- v-for 是一个列表渲染指令，它的作用是基于一个数组（或其他可迭代对象），重复渲染一个元素或组件多次。
	- 在实际开发中，`v-for` 通常应与 `:key` 属性一起使用，例如：`<li v-for="todo in todos" :key="todo.id">`。`key` 的作用是帮助 Vue 精确追踪每个节点的唯一身份，从而在列表数据发生变化时，只对必要的部分进行更新，最大限度地减少 DOM 操作并提升渲染性能。
- `{{ ... }}`
	- 文本插值表达式，它用于将 JavaScript 表达式的结果嵌入到 HTML 文本内容中。
	- **单向绑定**：它是单向的，只负责将数据从 JavaScript 状态流向 HTML 视图。
- **属性绑定：**
	- `:checked`
		- `:` 是 `v-bind` 指令的缩写。
		- **`:checked="todo.completed"`** 是 **`v-bind:checked`** 的缩写。
		- 用于将 数据 绑定到 HTML 元素的 属性。
		- `v-bind` 能够将 HTML 元素的属性与 Vue 的状态数据相连接，从而实现**单向数据绑定**（数据从模型流向视图）。
- **事件处理：**
	- `@click`
		- 是 `v-on:click` 的简写，绑定点击事件，点击之后会执行指定的函数。
	- `@change`
		- 是 **`v-on:change`** 的缩写。
		- 用于监听用户的交互事件并调用相应的方法。
- **条件渲染：**
	- `:class`
		- `:class="{ completed: todo.completed }"`
			- 当 `todo.completed` 为 $\text{true}$ 时，`completed` 类名会被应用。
- **`<style scoped>`**：
	- 它确保 CSS 样式只应用于 **当前组件** 的元素，避免了样式冲突，这在大型前端项目中非常重要。
- Composition API
	- `ref`
		- `ref`（Reference）函数用于创建响应式数据。
		- **作用 1：**用于存储组件的响应式状态。
		- **作用 2：**模板引用。
			- ref 也可以用来获取对 底层 DOM 元素 或 子组件实例 的直接引用。
			- **HTML 部分**：给元素添加 $\text{ref}$ 属性。
				- ```HTML
				  <input ref="inputRef" />
				  ```
			- **JavaScript 部分**：定义一个同名的 $\text{ref}$ 变量。
				- ```javascript
				  import { ref, onMounted } from 'vue'
				  
				  const inputRef = ref(null) // 初始值通常为 null
				  
				  onMounted(() => {
				    // 在组件挂载后，inputRef.value 会指向该 <input> DOM 元素
				    inputRef.value.focus() 
				  })
				  ```
		- **核心机制**
			- **包裹值（Wrapping Value）**：`ref` 接受一个值作为参数，将原始数据（如数字、字符串、布尔值、数组或对象等）包裹成一个引用对象返回，这个对象只有一个属性：`value`。
			- **追踪变化（Tracking Changes）**：通过该对象的 `.value` 属性访问或修改内部值时，Vue 会拦截这些操作，从而实现对数据变化的追踪。
			- **触发更新（Triggering Updates）**：当 `.value` 的值被修改后，Vue 会自动通知所有依赖该 `ref` 的组件或副作用函数（如 `watch`、`computed`），促使它们重新渲染或重新计算，以确保 UI 与数据保持同步。
		- **示例：**
			- ```JavaScript
			  import { ref } from 'vue'
			  
			  const count = ref(0) // count 是一个 ref 引用对象
			  console.log(count)         // 打印 { value: 0 }
			  console.log(count.value)   // 访问内部值：0
			  
			  // 修改值时必须通过 .value
			  count.value = 10 
			  console.log(count.value)   // 10
			  // 此时，所有使用 count 的组件都会自动更新
			  ```
		- **用 const 来接收 ref 的返回值**
			- 在 JavaScript 中，const 关键字并不是保证内容不可变，而是保证变量的绑定（Binding）不可变。
			- ```javascript
			  const count = ref(0)
			  ```
				- **`count` 这个变量名永远指向同一个 ref 对象**。你不能重新给 `count` 赋值。
				- 但是，你完全可以修改这个 `ref` 对象内部的属性 `.value`。
			- **好处：**这样可以避免对 `count` 变量的重新赋值，从而确保 **`count` 始终是一个 `ref` 引用对象**。如果不小心将 `count` 重新赋值为一个普通的非响应式值，它就会失去 `ref` 特性，变成普通的数值。此时，所有依赖 `count` 的组件都将无法感知变化，UI 也不会再自动更新。
			- 固定住装水的容器，但是容器内部装的是水还是蜂蜜无所谓。
		- **`.value` 的访问规则：**
			- 在 `<script>` 中，必须通过 `count.value` 来读取或修改内部值。
			- 而在 `<template>`（模板）中，Vue 会自动对 `ref` 进行深层解包（Unwrap），因此可以直接使用 `count`，无需手动访问 `count.value`。
		- **深度响应式：**
			- 如果你替换整个对象，会触发更新。
			- 如果你只修改对象内部的某个属性，或数组中的某个元素，也会触发更新。
			- **示例：**
				- ```javascript
				  const user = ref({ 
				    name: 'Alice', 
				    details: { age: 25 } 
				  })
				  
				  // 触发更新：修改深层属性
				  user.value.details.age = 26 
				  
				  // 触发更新：替换整个对象
				  user.value = { name: 'Bob' }
				  ```
			- **避免深度响应式**：如果你的对象或数组非常大，且你不需要对它的深层属性进行追踪，可以使用 `shallowRef()` 来替代，以优化性能。
-