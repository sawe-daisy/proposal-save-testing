```
proposeProject = async (
  oracleUtxo: UTxO,
  tokenUtxo: UTxO,
  memberUtxo: UTxO,
  fundRequested: number,
  receiver: string,
  metadata: ProposalMetadata
) => {
  // ADD DEBUG LOGGING FIRST
  console.log("=== DEBUG PROPOSEPROJECT ===");
  console.log("1. Oracle UTxO address:", oracleUtxo.output.address);
  console.log("2. Member UTxO address:", memberUtxo.output.address);
  console.log("3. Token UTxO address:", tokenUtxo.output.address);
  
  // Check member token in both UTxOs
  const memberTokenPolicyId = this.catConstant.scripts.member.mint.hash;
  console.log("4. Member token policy ID:", memberTokenPolicyId);
  
  // Check tokenUtxo contains member token
  const hasMemberTokenInTokenUtxo = tokenUtxo.output.amount.some(asset => 
    asset.unit.includes(memberTokenPolicyId)
  );
  console.log("5. Token UTxO contains member token?", hasMemberTokenInTokenUtxo);
  
  // Check memberUtxo contains member token (should be at script address)
  const hasMemberTokenInMemberUtxo = memberUtxo.output.amount.some(asset => 
    asset.unit.includes(memberTokenPolicyId)
  );
  console.log("6. Member UTxO contains member token?", hasMemberTokenInMemberUtxo);
  
  // Check addresses
  const memberScriptAddress = this.catConstant.scripts.member.spend.address;
  const isMemberUtxoAtScriptAddress = memberUtxo.output.address === memberScriptAddress;
  console.log("7. Member UTxO at script address?", isMemberUtxoAtScriptAddress);
  console.log("   Expected:", memberScriptAddress);
  console.log("   Actual:", memberUtxo.output.address);
  
  if (!hasMemberTokenInMemberUtxo) {
    console.error("ERROR: Member UTxO doesn't contain member token!");
    console.error("Member UTxO assets:", memberUtxo.output.amount);
  }
  
  if (!hasMemberTokenInTokenUtxo) {
    console.error("ERROR: Token UTxO doesn't contain member token!");
    console.error("Token UTxO assets:", tokenUtxo.output.amount);
  }
  
  if (!isMemberUtxoAtScriptAddress) {
    console.error("ERROR: Member UTxO not at script address!");
  }
  
  const memberAssetName = hexToString(
    getTokenAssetNameByPolicyId(
      memberUtxo,
      this.catConstant.scripts.member.mint.hash
    )
  );
  console.log("8. Extracted member asset name:", memberAssetName);
  
  const redeemer: ProposeProject = proposeProject(
    fundRequested,
    receiver,
    Number(memberAssetName),
    metadata
  );
  const datum: ProposalDatum = proposalDatum(
    fundRequested,
    receiver,
    Number(memberAssetName),
    metadata
  );
  const intentAssetName = computeProposalMetadataHash(metadata);
  
  console.log("9. Intent asset name:", intentAssetName);

  try {
    const txBuilder = await this.newValidationTx(true);
    txBuilder
      .readOnlyTxInReference(
        oracleUtxo.input.txHash,
        oracleUtxo.input.outputIndex,
        0
      )
      .readOnlyTxInReference(
        memberUtxo.input.txHash,
        memberUtxo.input.outputIndex,
        0
      )
      .txIn(
        tokenUtxo.input.txHash,
        tokenUtxo.input.outputIndex,
        tokenUtxo.output.amount,
        tokenUtxo.output.address,
        0
      )

      .mintPlutusScriptV3()
      .mint(
        "1",
        this.catConstant.scripts.proposeIntent.mint.hash,
        intentAssetName
      )
      .mintTxInReference(
        this.catConstant.refTxInScripts.proposeIntent.mint.txHash,
        this.catConstant.refTxInScripts.proposeIntent.mint.outputIndex,
        (
          this.catConstant.scripts.proposeIntent.mint.cbor.length / 2
        ).toString(),
        this.catConstant.scripts.proposeIntent.mint.hash
      )
      .mintRedeemerValue(redeemer, "JSON")

      .txOut(this.catConstant.scripts.proposeIntent.spend.address, [
        {
          unit:
            this.catConstant.scripts.proposeIntent.mint.hash +
            intentAssetName,
          quantity: "1",
        },
      ])
      .txOutInlineDatumValue(datum, "JSON");

    if (tokenUtxo.output.plutusData) {
      txBuilder
        .txOut(tokenUtxo.output.address, tokenUtxo.output.amount)
        .txOutInlineDatumValue(tokenUtxo.output.plutusData, "CBOR");
    } else {
      txBuilder.txOut(tokenUtxo.output.address, tokenUtxo.output.amount);
    }

    const txHex = await txBuilder.complete();
    console.log("10. Transaction built successfully!");

    const signedTx = await this.wallet.signTx(txHex, true);
    await this.wallet.submitTx(signedTx);

    return {
      txHex,
      proposeIntentUtxoTxIndex: 0,
      tokenUtxoTxIndex: 1,
    };
  } catch (e) {
    console.error("Error in proposeProject:", e);
    throw e;
  }
};
```