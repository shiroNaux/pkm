---
---
# Single Sign On
## Introduction

### Glossaries
- SSO:
- IdP:
- SP
### Abstraction
- Single sing on là cơ chế để đăng nhập nhiều dịch vụ khác nhau thông qua 1 service duy nhất.
- SSO chủ yếu thực hiện tác vụ [[Authentication]] chứ không bao gồm [[Authorization]] trong nó
- SSO có thể bị nhầm lẫn với Same sign on -> tức là dùng chung 1 credential cho mọi site (ví dụ: dùng chung username, password cho mọi web khác nha)
- SSO có thể trở thành SPOF khi mọi service khác yêu cầu user đăng nhập qua Identity Provider

### SSO flow
![[typical-sso-v2.png]]

Các bước thực hiện quá trình SSO
1. Người dùng gửi request đến dịch vụ (Service Provider / SP)
2. SP kiểm tra user đã đăng nhập hay chưa (thông qua token, cookies, ...), nếu chưa thì sẽ redirect user đó đến trang web của Identity Provider để đăng nhập
3. Người dùng sẽ thực hiện qua Identity provider
4. Khi login thành công, Identity provider sẽ redirect user sang 1 callback [[url]], url này sẽ trỏ đến SP. URL này sẽ kèm theo 1 số thông tin liên quan đến user
5. Khi SP nhận được URL callback này -> hiểu rằng user đã đăng nhập thành công, đồng thời nhận thông tin của user qua callback này để thực hiện [[Authorization]]

## 2. Implement SSO
Một số protocol thường dùng để implement SSO là
- [[OIDC|OpenID Connect]]
- [[LDAP]]
- [[SAML]]
- Kerberos
- Smart card

SSO là 1 thành phần của FIM. Hình ảnh dưới minh họa mối quan hệ giữa các khái niệm liên quan

![[sso-types.png]]
- Có thể thấy OAuth2 thực hiện cả 2 chức năng Authentication và Authorization. Ở khía cạnh Authen, OAuth2 sử dụng OpenID Connect để thực hiện đăng nhập

# Method & Protocol

| Feature\Method                  | Kerberos        | OIDC                       | LDAP                                                               | SAML | NTLM    | RADIUS  |
| ------------------------------- | --------------- | -------------------------- | ------------------------------------------------------------------ | ---- | ------- | ------- |
| Network Authentication Protocol |                 |                            |                                                                    |      |         |         |
| Primary Use                     | Authentication  | Authentication  + Identity |                                                                    |      |         |         |
| Architecture                    |                 |                            |                                                                    |      |         |         |
| Auth Flow type                  |                 |                            |                                                                    |      |         |         |
| SSO                             | Yes             | Yes                        | __No__ (LDAP is not protocol for SSO, just a part of SSO solution) | Yes  | Limited | Limited |
| Mutual Authen tication          |                 |                            |                                                                    |      |         |         |
| Token Type                      | Ticker (Binary) |                            |                                                                    |      |         |         |
| Cross Domain Support            |                 |                            |                                                                    |      |         |         |
| Complexity                      |                 |                            |                                                                    |      |         |         |
| Encryption & Security           |                 |                            |                                                                    |      |         |         |
|                                 |                 |                            |                                                                    |      |         |         |

## RADIUS

RADIUS is most appropriate for controlling network access for remote users, including those connecting via VPNs, Wi-Fi, or dial-up.

## NTLM

NTLM is most appropriate for maintaining compatibility with older Windows systems and applications that do not support more modern authentication protocols.

# Recommendation

- [[Kerberos]] & NTLM: Internal authen, Windows environment
- [[OIDC]]: Web based app, federated login
- [[LDAP]]: Hierarchical organizational data, on-prem
- RADIUS: Legacy, for network devices
- [[SAML]]: 

# References
1. [What Is and How Does Single Sign-On Authentication Work? (auth0.com)](https://auth0.com/blog/what-is-and-how-does-single-sign-on-work/)
2. [How Does Single Sign-On (SSO) Work? | OneLogin](https://www.onelogin.com/learn/how-single-sign-on-works)