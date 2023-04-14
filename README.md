# VMUG
## Môi trường demo

* VCSA 7.0.0.10400

## Điều kiện tiên quyết
Phần này mô tả cách sử dụng nó trên dòng lệnh.

* Python3.8 trở lên được cài đặt trên Linux (Ansible host)
* Cài đặt lệnh git
* Môi trường NetBox được xây dựng
    * Cluster Types và Clusters được khai báo
        * Clusters: phải giống với cụm trong môi trường vSphere
    * Mã thông báo có thể được vận hành với API được tạo
## Cách sử dụng

### Tạo venv

Tạo một Env ảo Python

```
$ python3 -m venv venv
$ . venv/bin/activate
(venv)$
```

### Nhận repo

```
(venv)$ git clone https://github.com/sky-joker/VMUG20220727.git
(venv)$ cd VMUG20220727
```

### Cài gói yêu cầu

```
pip install -r requirements.txt
ansible-galaxy collection install -r collections/requirements.yml -p collections
```

### Cài đặt tham số

Sửa đổi các tham số trong các tệp sau để phù hợp với môi trường của bạn.

```
vi inventories/group_vars/all.yml
```

|         Tên biến    |Kiểu|              Ví dụ               |                                                             Giải thích                                                             |
|---------------------|------|-------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| netbox_url          | str  | `http://netbox.local`         | Đường dẫn của netbox url                                                                   |
| netbox_token        | str  | 7182c49xxxxxxxxxxxxx          | NetBox Token                                                              |
| tenant_name         | str  | suncloud                       | Tên người thuê để tạo                                                                       |
| prefix              | str  | 10.0.11.0/24               | Prefix để tạo trong NetBox                                                                   |
| gateway             | str  | 10.0.11.1/24               | Default gateway của prefix                                           |
| vcenter_hostname    | str  | vcenter.vlware.local            | Tên máy chủ của vCenter Server để truy cập                                                     |
| vcenter_username    | str  | `administrator@vsphere.local` |Tài khoản máy chủ vCenter                                                     |
| vcenter_password    | str  | password                      |Mật khẩu tài khoản máy chủ ド                                                     |
| cluster_name        | str  | Cluster                       | Tên cụm để triển khai máy ảo (không thể sử dụng với esxi_hostname)(Nếu có cluster ko khai báo esxi_hostname)                        |
| esxi_hostname       | str  | esxi-01.local                 |Tên máy chủ ESXi để triển khai máy ảo (không thể sử dụng với cluster_name) (Nếu có esxi_hostname ko khai báo cluster_name)                      |
| datacenter_name     | str  | DC                            | Tên trung tâm dữ liệu nơi VM sẽ được triển khai |
| template_name       | str  | RHEL8.5TMP                    |Template sẽ dùng để clone ra các máy ảo    |
| dns_ip_addrs        | list |                               |DNS được đặt cho VM mới được tạo  |
| password            | str  | password                      | Mật khẩu root được đặt cho VM mới được tạo  |
| portgroup_name      | str  | pg01                          | Tên portgroup để kết nối với VM mới tạo                       |
| vm_size             | str  | 1cpu_2gmem                    | Kích thước VM mới được tạo, [chi tiết](roles/vmware_deploy_vm/defaults/main.yml) |
| count_number        | int  | 1                             | Số lượng máy ảo mới được tạo            |
| delete_ipaddr_regex | str  | 192.168.20.[1-5]              | Địa chỉ IP của VM sẽ bị xóa (cho phép biểu thức chính quy)                  |

```yaml
---
# Netbox
# netbox_url: http://example.com
# netbox_token:
# tenant_name:
# prefix: 192.168.20.0/24
# gateway: 192.168.20.1/24

# VMware
# vcenter_hostname:
# vcenter_username: administrator@vsphere.local
# vcenter_password:
# cluster_name:
# esxi_hostname:
# datacenter_name:
# template_name:
# dns_ip_addrs:
#   - 192.168.20.1
# password: The password does set to a guest
# portgroup_name:
# vm_size: 1cpu_2gmem

# Other Params
# count_number: 2
# delete_ipaddr_regex: 192.168.20.[2-3]
```

### Thực thi Playbook

#### Tạo Tenant và Prefix trên NetBox

Thực hiện lệnh sau.

```
(venv)$ ansible-playbook playbooks/netbox_add_prefix_menu.yml -i inventories/inventory
```
- Add a tenant `Suncloud` + add prefix `10.0.11.0/24` (sử dụng Ansible lấy Network) + Và add địa chỉ `10.0.11.1` là địa chỉ IP-gateway 
- Thực hiện add prefix `10.0.11.0/24` vào prefix Netbox
#### Deloy VM

Thực hiện lệnh sau.

```
(venv)$ ansible-playbook playbooks/deploy_vm_all_tasks.yml -i inventories/inventory
```
- `netbox_activate_ip_addr`:
    - `pre_check.yml`:  #Lấy ra được Prefix của tenant(Netbox)
    - `activate_ip_addr.yml`: #Lấy ra 1 địa chỉ available của prefix, Thêm địa chỉ IP đó vào Netbox
    - `set_aap_jobtemplate_variable.yml`
- `vmware_deploy_vm`:
    - `pre_check.yml`: Kiểm tra Cluster có tồn tại hay không.
    - `add_vm_folder.yml`: Tạo folder theo tenant
    - `deploy_vm_to_esxi.yml`: Kiểm tra nếu Cluster không có thì tạo VM vào `esxi_hostname`
    - `deploy_vm_to_cluster.yml`: Nếu có cluster thì thực hiện role này.
- 
#### Xóa VM

Thực hiện lệnh sau.

```
(venv)$ ansible-playbook playbooks/delete_vm_all_tasks.yml -i inventories/inventory
```
   