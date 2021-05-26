---
description: 动态数据掩码（Dynamic Data Masking，简称为DDM）能够防止把敏感数据暴露给未经授权的用户。
---

# 动态数据掩码

```text
| 类型 | 要求 | 示例 | 说明
| ---- | ---- | ---- | ---- 
| 手机号 | 前 3 后 4 | 132****7986 | 定长 11 位数字
| 邮箱地址 | 前 1 后 1 | l**w@gmail.com | 仅对 @ 之前的邮箱名称进行掩码
| 姓名 | 隐姓 | *鸿章 | 将姓氏隐藏
| 密码 | 不输出 | ****** | 
| 银行卡卡号 | 前 6 后 4 | 622888******5676 | 银行卡卡号最多 19 位数字
| 身份证号 | 前 1 后 1 | 1******7 | 定长 18 位
```

```text
package ddm

// 手机号 132****7986
type Mobile string

// 银行卡号 622888******5676
type BankCard string

// 身份证号 1******7
type IDCard string

// 姓名 *鸿章
// TODO:参考 https://blog.thinkeridea.com/201910/go/efficient_string_truncation.html
// Deprecated:有更好的性能选择
type Name string

// 密码 ******
type PassWord string

// 邮箱 l***w@gmail.com
type Email string
```

```text
package ddm

import (
   "fmt"
   "strings"
)

func (m Mobile) MarshalJSON() ([]byte, error) {
   if len(m) != 11 {
      return []byte(`"` + m + `"`), nil
   }

   v := fmt.Sprintf("%s****%s", m[:3], m[len(m)-4:])
   return []byte(`"` + v + `"`), nil
}

func (bc BankCard) MarshalJSON() ([]byte, error) {
   if len(bc) > 19 || len(bc) < 16 {
      return []byte(`"` + bc + `"`), nil
   }

   v := fmt.Sprintf("%s******%s", bc[:6], bc[len(bc)-4:])
   return []byte(`"` + v + `"`), nil
}

func (card IDCard) MarshalJSON() ([]byte, error) {
   if len(card) != 18 {
      return []byte(`"` + card + `"`), nil
   }

   v := fmt.Sprintf("%s******%s", card[:1], card[len(card)-1:])
   return []byte(`"` + v + `"`), nil
}

func (name Name) MarshalJSON() ([]byte, error) {
   if len(name) < 1 {
      return []byte(`""`), nil
   }

   nameRune := []rune(name)
   v := fmt.Sprintf("*%s", string(nameRune[1:]))
   return []byte(`"` + v + `"`), nil
}

func (pw PassWord) MarshalJSON() ([]byte, error) {
   v := "******"
   return []byte(`"` + v + `"`), nil
}

func (e Email) MarshalJSON() ([]byte, error) {
   if !strings.Contains(string(e), "@") {
      return []byte(`"` + e + `"`), nil
   }

   split := strings.Split(string(e), "@")
   if len(split[0]) < 1 || len(split[1]) < 1 {
      return []byte(`"` + e + `"`), nil
   }

   v := fmt.Sprintf("%s***%s", split[0][:1], split[0][len(split[0])-1:])
   return []byte(`"` + v + "@" + split[1] + `"`), nil
}
```

```text
package main

import (
	"encoding/json"
	"fmt"
	"gin-project/ddm"
)

type message struct {
	Name      ddm.Name     `json:"name"`
	Mobile    ddm.Mobile   `json:"mobile"`
	IDCard    ddm.IDCard   `json:"id_card"`
	PassWord  ddm.PassWord `json:"password"`
	Email     ddm.Email    `json:"email"`
	BankCard1 ddm.BankCard `json:"bank_card_1"`
	BankCard2 ddm.BankCard `json:"bank_card_2"`
	BankCard3 ddm.BankCard `json:"bank_card_3"`
}

func TestMarshalJSON() {
	msg := new(message)
	msg.Name = ddm.Name("李鸿章")
	msg.Mobile = ddm.Mobile("13288887986")
	msg.IDCard = ddm.IDCard("125252525252525252")
	msg.PassWord = ddm.PassWord("123456")
	msg.Email = ddm.Email("xinliangnote@163.com")
	msg.BankCard1 = ddm.BankCard("6545654565456545")
	msg.BankCard2 = ddm.BankCard("65485269874569852")
	msg.BankCard3 = ddm.BankCard("6548526987456985298")

	marshal, _ := json.Marshal(msg)
	fmt.Println(string(marshal))
}

func main() {
	TestMarshalJSON()
}
```

```text
{"name":"*鸿章","mobile":"132****7986","id_card":"1******2","password":"******","email":"x***e@163.com","bank_card_1":"654565******6545","bank_card_2":"654852******9852","bank_card_3":"
654852******5298"}
```

