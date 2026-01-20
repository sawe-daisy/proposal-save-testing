// Add this simple debug\
console.log("=== SIMPLE DEBUG ===");\
console.log("UserActionTx class:", UserActionTx);\
console.log("userAction instance:", userAction);

// Try to see the method\
console.log("proposeProject method exists?", typeof userAction.proposeProject);

// Call it and see what happens\
const result = await userAction.proposeProject(...);\
console.log("Result txHex length:", result.txHex?.length);