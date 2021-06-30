# 写文件

```go
package write

import (
	"github.com/cilidm/base-spider/app/util/zapLog"
	"os"
)

func WriteToFile(name string, content string, append bool) error {
	var fileObj *os.File
	var err error
	if append {
		fileObj, err = os.OpenFile(name, os.O_RDWR|os.O_CREATE|os.O_APPEND, 0644)
	} else {
		fileObj, err = os.OpenFile(name, os.O_RDWR|os.O_CREATE|os.O_TRUNC, 0644)
	}
	if err != nil {
		zapLog.NewLog().Error("WriteToFile", "os.OpenFile", err.Error())
		return err
	}
	defer fileObj.Close()
	if _, err := fileObj.WriteString(content); err != nil {
		zapLog.NewLog().Error("WriteToFile", "WriteString", err.Error())
		return err
	}
	return nil
}

```

