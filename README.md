# ansible-role-wireguard

Install and configure WireGuard VPN server.

## Requirements

- Debian/Ubuntu
- Collection: `ansible.posix`

## Supported Distributions

- Debian (bullseye, bookworm)
- Ubuntu (focal, jammy, noble)

Other distributions will fail with an error message.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `wireguard_interface` | `wg0` | Interface name |
| `wireguard_port` | `51820` | Listen port |
| `wireguard_address` | `10.0.0.1/24` | VPN subnet and server IP |
| `wireguard_private_key` | `""` | Private key (auto-generate if empty) |
| `wireguard_ip_forward` | `true` | Enable IP forwarding |
| `wireguard_dns` | `""` | DNS for clients |
| `wireguard_force_config` | `false` | Force config deployment (removes peers!) |

## Private Key Handling

The role handles private keys in three ways:

1. **Auto-generate** (recommended): Leave `wireguard_private_key` empty
   - Key is generated on first run
   - Stored in `/etc/wireguard/<interface>_privatekey`
   - Persists across role runs

2. **Provide manually**: Set `wireguard_private_key` in vault

```yaml
   wireguard_private_key: "{{ vault_wireguard_private_key }}"
```

3. **Existing key**: If key file already exists, it will be used

## Config Protection

The role will **NOT** overwrite WireGuard config if peers already exist.
This protects peer configurations added via [WGDashboard](https://github.com/WGDashboard/WGDashboard).

### Check status

When running the role, you'll see:

```
WireGuard config has 3 peer(s). Config will NOT be modified.
Skipping config deployment - 3 peer(s) found. Manage peers via [WGDashboard](https://github.com/WGDashboard/WGDashboard).
```

### Force config update (WARNING!)

**Dangerous! Will remove all peers!!!**

```yaml
wireguard_force_config: true
```

## Files Created

| File | Description |
|------|-------------|
| `/etc/wireguard/<interface>.conf` | WireGuard configuration |
| `/etc/wireguard/<interface>_privatekey` | Private key (mode 0600) |
| `/etc/wireguard/<interface>_publickey` | Public key |

## Examples

### Basic usage

```yaml
wireguard_interface: "wg0"
wireguard_address: "10.10.0.1/24"
wireguard_port: 51820
```

### Custom interface name

```yaml
wireguard_interface: "wg-pl-01"
wireguard_address: "10.10.1.1/24"
```

### Custom subnet with DNS

```yaml
wireguard_interface: "wg0"
wireguard_address: "172.16.0.1/24"
wireguard_dns: "1.1.1.1, 8.8.8.8"
```

## NAT Configuration

NAT is now managed by the `firewall` role instead of WireGuard PostUp/PostDown scripts.
Enable WireGuard NAT in the firewall role:

```yaml
firewall_wireguard_enabled: true
firewall_wireguard_interface: "wg0"
firewall_wireguard_network: "10.0.0.0/24"
```

## Output

During deployment, the role displays:

```
TASK [wireguard : Display peers status]
ok: [wg-pl-01] => msg: "WireGuard config has 0 peer(s). Config will be deployed."

TASK [wireguard : Display public key]
ok: [wg-pl-01] => msg: "WireGuard public key: ABC123..."
```

Save the public key for client configurations.

## Notes

- Peers are managed via [WGDashboard](https://github.com/WGDashboard/WGDashboard) (see ansible-role-wgdashboard)
- Public key is displayed during deployment for client configuration
- Config is protected from overwriting if peers exist
- NAT interface is auto-detected if not specified

## License

MIT
