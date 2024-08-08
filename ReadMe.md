## Giới thiệu
Đây là repo gồm các yêu cầu của mình cho ChatLLM (GPT-4 và Claude) để sinh code tự động

## Yêu cầu đầu tiên
Please code Go web app that use authorization RBAC to allow or deny HTTP request bases on  url path (/api/book, /api/person...), method (GET, POST, PUT, DELETE), and roles.
Authorization expression for each url path and method can be:

Allow(roleA, roleB): only roleA and roleB are allowed
Deny(roleB, roleC): all roles are allowed except roleB and roleC
AllowAll: allow all roles
DenyAll: deny all roles