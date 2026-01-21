```
console.error("=== ERROR CAUGHT ===");
    console.error("Error name:", error.name);
    console.error("Error message:", error.message);
    console.error("Error stack:", error.stack);
    
    // Check for specific patterns
    if (error.message.includes("unexpected empty list")) {
      console.error("DIAGNOSIS: Aiken validator error - missing required input");
    }
```