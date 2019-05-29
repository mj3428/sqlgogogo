## char与varchar
char是**固定长度**的，varchar是**可变长度**的，char速度快但是消耗存储空间；  
  - MyISAM引擎: 建议使用固定长度的数据列  
  - MEMORY引擎: 目前都使用固定长度的数据行存储，无论使用char还是varchar都是作为char类型处理；  
  - InnoDB引擎: 建议使用varchar类型。内部的行存储格式没有区分固定长度和可变长度列（所有数据行都使用指向数据列值的头指针）。  
  
