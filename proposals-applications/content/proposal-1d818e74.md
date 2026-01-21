```
const tokenUtxo = await findTokenUtxoByMemberUtxo(mbrUtxo);

if (!tokenUtxo) {
  // Debug what token it's looking for
  const datum: MemberDatum = deserializeDatum(mbrUtxo.output.plutusData);
  const policyId = datum.fields[0].list[0].bytes;
  const assetName = datum.fields[0].list[1].bytes;
  const tokenUnit = policyId + assetName;
  
  console.error("=== TOKEN NOT FOUND ===");
  console.error("Looking for token unit:", tokenUnit);
  console.error("Policy ID:", policyId);
  console.error("Asset name:", assetName);
  
  // Check your wallet for this EXACT token
  console.error("Your wallet assets:");
  walletUtxos.forEach(utxo => {
    utxo.output.amount.forEach(asset => {
      console.error("  -", asset.unit, "x", asset.quantity);
    });
  });
  
  throw new Error(
    `Required token not found in your wallet. ` +
    `You need token: ${policyId.substring(0, 16)}...${assetName.substring(0, 16)}... ` +
    `This is the token you used when applying for membership.`
  );
}
```