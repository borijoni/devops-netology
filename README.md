# все файлы во всех поддиректориях с вложенной в них дирректорией .terraform 
**/.terraform/*

# все файлы с расширением tfstate, с рекурсией
*.tfstate

# все файлы с любым расширением, начинающиеся с любого набора символов за которыми следует .tfstate, с рекурсией
*.tfstate.*

# файл crash.log
crash.log

# все файлы с расширением tfvars, с рекурсией
*.tfvars

# файл override.tf
override.tf

# файл override.tf.json
override.tf.json

# файлы с расширением tf, начинающиеся с любого набора символов за которыми следует _override
*_override.tf

# файлы с расширением json, начинающиеся с любого набора символов за которыми следует _override.tf
*_override.tf.json

# файл .terraformrc
.terraformrc

# файл terraform.rc
terraform.rc
New Line