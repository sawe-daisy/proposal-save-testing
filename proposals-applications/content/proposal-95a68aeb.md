## code@dee-thinkpad-t470-w10dg:\~/lidonation/cardano-ambassador-tool$ cd \~/lidonation/cardano-ambassador-tool/off-chain
grep -n "proposeProject" src/transactions/userAction.ts -A 50
8:  proposeProject,
9-  ProposalDatum,
10-  proposalDatum,
11-  IProvider,
12-  getTokenAssetNameByPolicyId,
13-  computeProposalMetadataHash,
14-  CATConstants,
15-  updateMembershipIntentMetadata,
16-  getMembershipIntentDatum,
17-  MemberDatum,
18-  memberDatum,
19-  getMemberDatum,
20-  memberUpdateMetadata,
21-  MembershipMetadata,
22-  ProposalMetadata,
23-} from "../lib";
24-import {
25-  deserializeAddress,
26-  hexToString,
27-  IWallet,
28-  mConStr1,
29-  UTxO,
30-} from "@meshsdk/core";
31-
32-export class UserActionTx extends Layer1Tx {
33-  constructor(
34-    public address: string,
35-    public userWallet: IWallet,
36-    public provider: IProvider,
37-    public catConstant: CATConstants
38-  ) {
39-    super(userWallet, address, provider, catConstant);
40-  }
41-
42-  /\*\*
43-   \*
44-   \* @param oracleUtxo
45-   \* @param tokenUtxo
46-   \* @param tokenPolicyId
47-   \* @param tokenAssetName assetName in hex
48-   \* @param walletAddress address in bech32
49-   \* @param metadata
50-   \* @returns
51-   \*/
52-  applyMembership = async (
53-    oracleUtxo: UTxO,
54-    tokenUtxo: UTxO,
55-    tokenPolicyId: string,
56-    tokenAssetName: string,
57-    metadata: MembershipMetadata
58-  ) =&gt; {

316:  proposeProject = async (
317-    oracleUtxo: UTxO,
318-    tokenUtxo: UTxO,
319-    memberUtxo: UTxO,
320-    fundRequested: number,
321-    receiver: string,
322-    metadata: ProposalMetadata
323-  ) =&gt; {
324-    const memberAssetName = hexToString(
325-      getTokenAssetNameByPolicyId(
326-        memberUtxo,
327-        this.catConstant.scripts.member.mint.hash
328-      )
329-    );
330:    const redeemer: ProposeProject = proposeProject(
331-      fundRequested,
332-      receiver,
333-      Number(memberAssetName),
334-      metadata
335-    );
336-    const datum: ProposalDatum = proposalDatum(
337-      fundRequested,
338-      receiver,
339-      Number(memberAssetName),
340-      metadata
341-    );
342-    const intentAssetName = computeProposalMetadataHash(metadata);
343-
344-    try {
345-      const txBuilder = await this.newValidationTx(true);
346-      txBuilder
347-        .readOnlyTxInReference(
348-          oracleUtxo.input.txHash,
349-          oracleUtxo.input.outputIndex,
350-          0
351-        )
352-        .readOnlyTxInReference(
353-          memberUtxo.input.txHash,
354-          memberUtxo.input.outputIndex,
355-          0
356-        )
357-        .txIn(
358-          tokenUtxo.input.txHash,
359-          tokenUtxo.input.outputIndex,
360-          tokenUtxo.output.amount,
361-          tokenUtxo.output.address,
362-          0
363-        )
364-
365-        .mintPlutusScriptV3()
366-        .mint(
367-          "1",
368-          this.catConstant.scripts.proposeIntent.mint.hash,
369-          intentAssetName
370-        )
371-        .mintTxInReference(
372-          this.catConstant.refTxInScripts.proposeIntent.mint.txHash,
373-          this.catConstant.refTxInScripts.proposeIntent.mint.outputIndex,
374-          (
375-            this.catConstant.scripts.proposeIntent.mint.cbor.length / 2
376-          ).toString(),
377-          this.catConstant.scripts.proposeIntent.mint.hash
378-        )
379-        .mintRedeemerValue(redeemer, "JSON")
380-
code@dee-thinkpad-t470-w10dg:\~/lidonation/cardano-ambassador-tool/off-chain$