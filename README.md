# Quasar-Supabase-E
An example Quasar app that connects to Supabase.io

This project was cloned from https://github.com/tequila99/quasar-supabase
and translated to English.

After downloading or cloning the directory:
1) Type npm install or yarn install
2) Create a .env file and add it to the top level dirctory with this content:

  VITE_SUPABASE_URL=<URL_from_your_supabase_account><br/>
  VITE_SUPABASE_ANON_KEY=<KEY_from_your_supabase_account>

3) Type Quasar dev

If the initial page is blank go to localhost:3000/auth/login

Changes:
1) Added MagicLink login option

Issues
1) Profile is not loaded after login

Notes:
1) Login with Github and Google need to be enabled in Supabase
2) If only an email address is provided when the registration is submitted MagicLink (email) login will be enabled. (Its not tested whether its possible to add a password later)

Any comments and suggestions to admin @ nodescan dot com
