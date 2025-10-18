-
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