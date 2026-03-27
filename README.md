# geoip-block.sh

Fetches aggregated IP ranges for one or more countries from [ipdeny.com](https://www.ipdeny.com) and writes iptables `DROP` rules to `/etc/iptables/rules.v4`, blocking all inbound traffic from those ranges on all ports.

## Requirements

- Root privileges
- `curl`
- `iptables`
- `netfilter-persistent` (Debian/Ubuntu: `apt install iptables-persistent`)

## Usage

Pass countries as a comma-separated environment variable or as space-separated arguments. Country codes follow the [ISO 3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) standard (e.g. `ru`, `cn`, `in`).

```bash
# Via environment variable
COUNTRIES=<country1>,<country2>,<country3> ./update-country-blocks.sh

# Via arguments
./update-country-blocks.sh <country1> <country2> <country3>
```

## What it does

1. Downloads `<country>-aggregated.zone` for each specified country from ipdeny.com.
2. Generates `/etc/iptables/rules.v4` from scratch, replacing any previous content.
3. Applies the rules immediately with `iptables-restore`.
4. Restarts `netfilter-persistent.service` so the rules persist across reboots.

Each rule includes an inline comment identifying the country, visible via `iptables -L INPUT -n`:

```
DROP  all  --  1.2.3.0/24  0.0.0.0/0  /* country-block:RU */
```

## Notes

- The script is **non-additive** — every run fully regenerates the rules file and reloads it, so there is no risk of duplicate rules accumulating.
- All existing rules in the `filter` table are replaced on each run. If you have other custom rules, merge them into the generated file or manage them in a separate chain.
- Only `INPUT` traffic is blocked. Adjust the `CHAIN` variable in the script if you also need to cover `FORWARD`.
