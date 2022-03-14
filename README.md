# RPC node & State sync endpoint for Umee Mainnet
# RPC node

RPC endpoint with "default" pruning: http://142.132.204.100:26657/ (hosted by wombat#7690)

# State sync guide

1. Follow official documentation in order to setup a full node in Umee Mainnet: https://docs.umee.cc/umee/umee-node-operators/running-a-node/joining-mainnet.
Make sure you use the latest version of ``umeed``.
2. Set up a service to run ``umeed binary``.
3. Make sure it is stopped and reset database:
```
sudo systemctl stop umeed.service
umeed unsafe-reset-all
```
4. Use variables to connect to RPC node and set the number and the hash of the block for state sync:
```
RPC_PEER="db5cb0abc778ced85a1fc9b510a3bd598c02a2e2@142.132.204.100:26656"
SNAP_RPC="http://142.132.204.100:26657"
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height)
TRUST_HEIGHT=$((LATEST_HEIGHT - 1000))
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$TRUST_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$TRUST_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.umee/config/config.toml
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$RPC_PEER\"/" $HOME/.umee/config/config.toml
```
5. Run the service and wait for a few minutes until snaphost is fetched and applied:
```
sudo systemctl restart umeed.service && sudo journalctl -u umeed.service -f -o cat
```
