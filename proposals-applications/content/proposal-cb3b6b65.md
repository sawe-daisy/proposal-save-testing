code@dee-thinkpad-t470-w10dg:\~/lidonation/cardano-ambassador-tool/client$ docker run --rm -p 3000:3000 --env-file .env.local cat-client

&gt; cardano-ambassador-tool@0.1.0 start

&gt; next start

   ▲ Next.js 15.5.9

   - Local:        <http://localhost:3000>

   - Network:      <http://172.17.0.2:3000>

 ✓ Starting...

 ⚠ "next start" does not work with "output: standalone" configuration. Use "node .next/standalone/server.js" instead.

 ✓ Ready in 890ms

\[dotenv@17.2.3\] injecting env (0) from .env.local -- tip: 🛠️  run anywhere with `dotenvx run -- yourcommand`

\[dotenv@17.2.3\] injecting env (0) from .env -- tip: 🔄 add secrets lifecycle management: <https://dotenvx.com/ops>

{

  item: { k: { constructor: 0n, fields: \[Array\] }, v: { int: 10000000n } }

}

{

  item: { k: { constructor: 0n, fields: \[Array\] }, v: { int: 10000000n } }

}

Storage API error: Error \[StorageApiError\]: Failed to get file: Could not load credentials from any providers

    at h.get (.next/server/app/api/storage/route.js:1:16483)

    at async w (.next/server/app/api/storage/route.js:1:1651)

    at async k (.next/server/app/api/storage/route.js:1:4963)

    at async g (.next/server/app/api/storage/route.js:1:5966)

    at async C (.next/server/app/api/storage/route.js:1:7088) {

  status: 500

}