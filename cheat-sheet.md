---
icon: hand-point-right
---

# Cheat sheet

### Service operations <a href="#service-operations" id="service-operations"></a>

```
# Reload Service
sudo systemctl daemon-reload

# Enable Service
sudo systemctl enable symphonyd

# Disable Service
sudo systemctl disable symphonyd

# Start Service
sudo systemctl start symphonyd

# Stop Service
sudo systemctl stop symphonyd

# Restart Service
sudo systemctl restart symphonyd

# Check Service Status
sudo systemctl status symphonyd

# Check Service Logs
sudo journalctl -u symphonyd -f --no-hostname -o cat
```

Wallet

```
# Add New Wallet
symphonyd keys add wallet

# Restore executing wallet
symphonyd keys add wallet --recover

# List All Wallets
symphonyd keys list

# Delete wallet
symphonyd keys delete wallet

# Check Balance
symphonyd q bank balances $(symphonyd keys show wallet -a)

# Export Key (save to wallet.backup)
symphonyd keys export sedad

# View EVM Prived Key
symphonyd keys unsafe-export-eth-key sedad

# Import Key (restore from wallet.backup)
symphonyd keys import sedad wallet.backup
```
