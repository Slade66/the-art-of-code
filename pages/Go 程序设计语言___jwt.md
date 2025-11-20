- `func NewNumericDate(t time.Time) *NumericDate`
	- **作用：**此函数用于将 Go 语言标准库中的 `time.Time` 类型对象，转换为符合 JWT 标准要求的 `NumericDate` 格式。
	- **为什么需要 NumericDate？**
		- 在 JWT 标准中，所有与时间相关的声明（如 `exp` 过期时间、`iat` 签发时间等）都必须使用 `NumericDate` 来表示。
		- `NumericDate` 是一种特定的数字时间戳，它是一个整数，代表自 Unix 纪元（1970 年 1 月 1 日 00:00:00 UTC）以来经过的秒数。
- `func NewWithClaims(method SigningMethod, claims Claims, opts ...TokenOption) *Token`
	- **作用：**使用指定的签名算法（如 HS256）和载荷数据（Claims）在内存中构建一个新的 Token 对象，并返回指向该对象的指针（注意：此时 Token 尚未签名生成字符串）。
- `func (t *Token) SignedString(key any) (string, error)`
	- **作用：**使用传入的密钥（Key）对 Token 进行数字签名，并输出最终生成的、完整的 JWT 字符串。
	- **工作流程：**
		- **生成待签名串：**先将 Token 的 Header 和 Claims 分别进行 Base64Url 编码，并用 `.` 连接，生成待签名的字符串 `sstr`。
		  logseq.order-list-type:: number
		- **计算数字签名：**使用 Token 中指定的签名算法和传入的密钥，对上一步生成的 `sstr` 进行加密签名，得到原始签名数据 `sig`。
		  logseq.order-list-type:: number
		- **编码并拼接：**将原始签名 `sig` 也进行 Base64Url 编码，最后拼接到 `sstr` 后面，形成最终的 JWT 字符串返回。
		  logseq.order-list-type:: number
-