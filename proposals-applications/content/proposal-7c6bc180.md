```
if (!memberUtxo) {
      console.error("ERROR: memberUtxo is null/undefined from context");
      throw new Error('No membership application UTxO found for this address');
    }
    
    console.log("2. memberUtxo from context:", memberUtxo);
    console.log("3. memberUtxo address:", memberUtxo.address);
    
    // --- MOVE THIS UP ---
    // Get constants EARLY so we can use memberTokenPolicyId
    const catConstants = getCatConstants();
    const memberTokenPolicyId = catConstants.scripts.member.mint.hash;
    console.log("4. Member token policy ID:", memberTokenPolicyId);
    
    const mbrUtxo = dbUtxoToMeshUtxo(memberUtxo);
    console.log("5. mbrUtxo after dbUtxoToMeshUtxo:", mbrUtxo);
    console.log("6. mbrUtxo output address:", mbrUtxo?.output?.address);
    console.log("7. mbrUtxo assets:", mbrUtxo?.output?.amount);
    
    if (!userWallet) {
      throw new Error('Wallet not connected');
    }

    console.log("=== FINDING MEMBER TOKEN IN WALLET ===");
    const walletUtxos = await userWallet.getUtxos();
    console.log("Wallet UTxOs count:", walletUtxos.length);

    // Log all wallet assets for debugging
    walletUtxos.forEach((utxo, index) => {
      console.log(`UTxO ${index}:`, {
        txHash: utxo.input.txHash,
        address: utxo.output.address,
        assets: utxo.output.amount.map(a => ({
          unit: a.unit,
          quantity: a.quantity
        }))
      });
    });

    // Find UTxO with MEMBER TOKEN
    const tokenUtxo = walletUtxos.find(utxo => {
      const hasToken = utxo.output.amount.some(asset => 
        asset.unit.includes(memberTokenPolicyId)  // NOW DEFINED!
      );
      return hasToken;
    });

    if (!tokenUtxo) {
      console.error("ERROR: No member token found in wallet!");
      console.error("Member token policy needed:", memberTokenPolicyId);
      console.error("Available assets in wallet:");
      walletUtxos.forEach(utxo => {
        utxo.output.amount.forEach(asset => {
          console.log("  -", asset.unit.substring(0, 64) + "...", "x", asset.quantity);
        });
      });
      
      throw new Error(
        `No member token found in your wallet. ` +
        `You need to have the member token (policy: ${memberTokenPolicyId.substring(0, 16)}...) ` +
        `in your wallet to submit proposals.`
      );
    }

    console.log("Found token UTxO with member token:", {
      txHash: tokenUtxo.input.txHash,
      address: tokenUtxo.output.address,
      assets: tokenUtxo.output.amount
    });

    const oracleUtxos = await blockfrost.fetchUTxOs(
      ORACLE_TX_HASH,
      ORACLE_OUTPUT_INDEX,
    );
    console.log("8. oracleUtxos fetched:", oracleUtxos.length);
    
    const oracleUtxo = oracleUtxos[0];
    console.log("9. oracleUtxo:", oracleUtxo);

    if (!oracleUtxo) {
      console.error("ERROR: No oracle UTxO found");
      throw new Error('Failed to fetch required oracle UTxO');
    }

    // Already have catConstants from above
    console.log("10. Cat Constants member script address:", catConstants.scripts.member.spend.address);
    
    // Check if mbrUtxo is at correct address
    const memberScriptAddress = catConstants.scripts.member.spend.address;
    const isMbrAtScriptAddress = mbrUtxo?.output?.address === memberScriptAddress;
    console.log("11. Is mbrUtxo at script address?", isMbrAtScriptAddress);
    console.log("   Expected:", memberScriptAddress);
    console.log("   Actual:", mbrUtxo?.output?.address);
    
    // Check if mbrUtxo contains member token (already have memberTokenPolicyId)
    const hasMemberTokenInMbr = mbrUtxo?.output?.amount?.some(asset => 
      asset.unit.includes(memberTokenPolicyId)
    );
    console.log("12. Does mbrUtxo contain member token?", hasMemberTokenInMbr);
    
    // Check if tokenUtxo contains member token
    const hasMemberTokenInToken = tokenUtxo?.output?.amount?.some(asset =>
      asset.unit.includes(memberTokenPolicyId)
    );
    console.log("13. Does tokenUtxo contain member token?", hasMemberTokenInToken);
    
    console.log("14. Creating UserActionTx...");

    const userAction = new UserActionTx(
      userAddress!,
      userWallet!,
      blockfrost,
      catConstants,
    );
```