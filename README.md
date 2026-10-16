# hommy-ad-revenue-hus<script type="module">
import { createClient } from
  'https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2/+esm';

// Supabase Dashboard se apna Project URL copy karein
const SUPABASE_URL = 'YOUR_SUPABASE_PROJECT_URL';

// Aapki Publishable Key
const SUPABASE_KEY =
  'sb_publishable_OQo9Nq9T4RnFVoSF2TT4RQ_VOP95YuJ';

const supabase = createClient(SUPABASE_URL, SUPABASE_KEY);

const status = document.getElementById('authStatus');
const signup = document.getElementById('signup');
const login = document.getElementById('login');
const logout = document.getElementById('logout');

async function refreshSession() {
  const { data, error } = await supabase.auth.getSession();

  if (error) {
    status.textContent = 'Session check failed. Please refresh.';
    return;
  }

  const user = data.session?.user;

  status.textContent = user
    ? 'Logged in: ' + user.email
    : 'Please sign up or log in.';

  signup.hidden = !!user;
  login.hidden = !!user;
  logout.hidden = !user;
}

signup.addEventListener('click', async () => {
  const name = document.getElementById('displayName').value.trim();
  const email = document.getElementById('email').value.trim();
  const password = document.getElementById('password').value;

  if (!name || !email || password.length < 8) {
    status.textContent =
      'Enter your name, valid email and password of at least 8 characters.';
    return;
  }

  signup.disabled = true;
  status.textContent = 'Creating account...';

  try {
    const { data, error } = await supabase.auth.signUp({
      email,
      password,
      options: { data: { display_name: name } }
    });

    if (error) throw error;

    status.textContent = data.session
      ? 'Account created! You are logged in.'
      : 'Account submitted! Check your email to verify it.';

    await refreshSession();
  } catch (error) {
    status.textContent = error.message || 'Signup failed.';
  } finally {
    signup.disabled = false;
  }
});

login.addEventListener('click', async () => {
  const email = document.getElementById('email').value.trim();
  const password = document.getElementById('password').value;

  login.disabled = true;
  status.textContent = 'Logging in...';

  try {
    const { error } = await supabase.auth.signInWithPassword({
      email,
      password
    });

    if (error) throw error;

    status.textContent = 'Login successful!';
    await refreshSession();
  } catch (error) {
    status.textContent = error.message || 'Login failed.';
  } finally {
    login.disabled = false;
  }
});

logout.addEventListener('click', async () => {
  const { error } = await supabase.auth.signOut();

  status.textContent = error
    ? error.message
    : 'You have logged out.';

  await refreshSession();
});

supabase.auth.onAuthStateChange(() => {
  refreshSession();
});

refreshSession();
</script>
