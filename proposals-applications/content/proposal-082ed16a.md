./src/app/my/proposals/\[txhash\]/page.tsx\
⨯ TypeError: Cannot read properties of undefined (reading 'length')\
at  (.next/server/app/api/txs/route.js:702:47)\
at Object. (.next/server/app/api/txs/route.js:705:3) {\
page: '/api/txs'\
}\
⨯ TypeError: Cannot read properties of undefined (reading 'length')\
at  (.next/server/app/my/submissions/page.js:1686:47)\
at Object. (.next/server/app/my/submissions/page.js:1689:3) {\
page: '/my/submissions'\
}\
POST /api/txs 500 in 9227ms\
GET /api/blockfrost/preprod/addresses/addr_test1wqdaqfxu06e2eeflt43dcevkwzlapvntm0pcfuh2fn6ggkqrt0jvp/utxos?page=2 200 in 7939ms\
POST /my/submissions 500 in 9413ms