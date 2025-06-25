# Rpcclient

## Without Crendentials

### Null Sesion Rpcclient Enum Users

```
rpcclient -U "" <DC_IP> -N -c "enumdomusers"
```

## With Crendentials

### Enumeration All Domain Users

```
rpcclient -U "deimcorp.local\champi%Password2" <AD_IP> -c "enumdomusers"
```

### Enumeration Description All Domain Users&#x20;

{% code overflow="wrap" %}
```bash
for rid in $(rpcclient -U "deimcorp.local\champi%Password2" <DC-IP> -c "enumdomusers" | grep -oP 'rid:\[0x[0-9a-fA-F]+\]' | tr -d '[]' | awk -F: '{print $2}'); do echo -e "\n[*] RID: $rid:\n"; rpcclient -U "deimcorp.local\champi%Password2" <DC-IP> -c "queryuser $rid" | grep -E -i "user name|description" ;done
```
{% endcode %}

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Enumeration All Admins Users

```bash
rpcclient -U "deimcorp.local\champi%Password2" <IP-DC>
-----------------------------------------------------------
rpcclient $> enumdomgroups #Enumerar todos los grupos existentes
    group:[Admins. del dominio] rid:[0x200]
rpcclient $> querygroupmem 0x200 # Filtrar solo por ese grupo
	rid:[0x1f4] attr:[0x7]
	rid:[0x453] attr:[0x7]
	rid:[0x454] attr:[0x7]
rpcclient $> queryuser 0x1f4 #Iteras por el rid
	User Name   :	Administrador
	Description :	Cuenta integrada para la administración del equipo o dominio
```
