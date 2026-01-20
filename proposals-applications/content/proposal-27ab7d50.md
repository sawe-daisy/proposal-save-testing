```
trace("Processing MintProposal redeemer")
    
    // ADD THIS DEBUG BEFORE THE PATTERN MATCH
    trace("Looking for intent at address: #{propose_intent_address}")
    trace("Looking for token with policy: #{propose_intent_token}")
    
    // Debug: Show all inputs
    trace("Total inputs: #{inputs |> length}")
    inputs |> list.index_map(fn(i, input) {
      trace("Input #{i}: address=#{input.output.address}")
      trace("Input #{i} assets: #{input.output.value |> Value.to_list()}")
    })
    
    // Try to find matching inputs first
    let possible_intents = inputs_at_with_policy(
      inputs,
      propose_intent_address,
      propose_intent_token,
    )
    
    trace("Found #{possible_intents |> length} possible intents")
```