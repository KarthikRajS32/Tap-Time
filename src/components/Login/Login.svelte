<script context="module" lang="ts">
  // Type declaration for Google Sign-In
  declare const google: {
    accounts: {
      id: {
        initialize: (config: {
          client_id: string;
          callback: (response: { credential: string }) => void;
          hosted_domain?: string;
          ux_mode?: 'popup' | 'redirect';
          auto_select?: boolean;
        }) => void;
        renderButton: (
          element: HTMLElement,
          options: { 
            theme?: 'outline' | 'filled_blue' | 'filled_black';
            size?: 'small' | 'medium' | 'large';
            width?: number;
            text?: 'signin_with' | 'signup_with' | 'continue_with';
            shape?: 'rectangular' | 'pill' | 'circle' | 'square';
            logo_alignment?: 'left' | 'center';
          }
        ) => void;
        prompt: (callback?: (notification: {isNotDisplayed: boolean, isSkipped: boolean}) => void) => void;
      };
    };
  };
</script>

<script lang="ts">
  import { onMount } from 'svelte';

  let username = '';
  let password = '';
  let errorMsg = '';
  let loading = false;
  let logoSrc = '';

  const key1 = new Uint8Array([
    16, 147, 220, 113, 166, 142, 22, 93,
    241, 91, 13, 252, 112, 122, 119, 95
  ]);

  onMount(() => {
    // Load company logo if exists
    const storedLogo = localStorage.getItem('companyLogo');
    if (storedLogo) {
      logoSrc = storedLogo;
    }

    // Load Google Sign-In script
    loadGoogleSignIn();
  });
 
  function loadGoogleSignIn() {
    // @ts-ignore
    if (window.google) {
      initializeGoogleSignIn();
      return;
    }

    const script = document.createElement('script');
    script.src = 'https://accounts.google.com/gsi/client';
    script.async = true;
    script.defer = true;
    script.onload = () => initializeGoogleSignIn();
    document.head.appendChild(script);
  }

  function initializeGoogleSignIn() {
    try {
      google.accounts.id.initialize({
        client_id: "1070255023214-gc25jf1quuc0bgu7vut9e2g4nghlhtbs.apps.googleusercontent.com",
        callback: handleCredentialResponse,
        auto_select: true,
      });

      // Render the button with customized appearance
      const buttonDiv = document.getElementById('googleSignInBtn');
      if (buttonDiv) {
        google.accounts.id.renderButton(buttonDiv, {
          theme: 'outline',
          size: 'large',
          width: 300, // Fixed width for consistency
          text: 'continue_with', // Matches your screenshot
          shape: 'rectangular',
          logo_alignment: 'left'
        });
        
        // Optional: Show the one-tap prompt
        google.accounts.id.prompt(notification => {
          if (notification.isNotDisplayed || notification.isSkipped) {
            // Fallback to button if prompt isn't shown
          }
        });
      }
    } catch (error) {
      console.error('Google Sign-In initialization failed:', error);
    }
  }

  // Handle Google Sign-In response
  async function handleCredentialResponse(response: { credential: string }) {
    errorMsg = '';
    loading = true;
    
    try {
      const userObject = decodeJwtResponse(response.credential);
      const email = userObject.email;

      const res = await fetch(`https://vnnex1njb9.execute-api.ap-south-1.amazonaws.com/test/login_check/${email}`);
      if (!res.ok) {
        throw new Error('Network response was not ok');
      }

      const data = await res.json();

      if ("error" in data) {
        errorMsg = "Invalid Gmail login";
        loading = false;
        return;
      }

      if (data["AdminType"] === "Admin" || data["AdminType"] === "SuperAdmin") {
        const companyID = data["CID"];
        localStorage.setItem('companyID', companyID);
        localStorage.setItem('companyName', data["CName"]);
        localStorage.setItem('companyLogo', data["CLogo"]);
        localStorage.setItem('companyAddress', data["CAddress"]);
        localStorage.setItem('username', data["UserName"]);
        localStorage.setItem('password', data["Password"]);
        localStorage.setItem('reportType', data["ReportType"]);
        localStorage.setItem('adminMail', data["Email"]);
        localStorage.setItem('adminType', data["AdminType"]);

        await Promise.all([
          getTimeZone(companyID),
          getCustomerData(companyID)
        ]);

        window.location.href = '/employeelist';
      }
    } catch (error) {
      console.error("Google Sign-In error:", error);
      errorMsg = 'An error occurred during Google Sign-In';
      loading = false;
    }
  }

  // Decode JWT token from Google
  function decodeJwtResponse(token: string) {
    const base64Url = token.split('.')[1];
    const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
    const jsonPayload = decodeURIComponent(
      atob(base64)
        .split('')
        .map((c) => '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2))
        .join('')
    );
    return JSON.parse(jsonPayload);
  }

  // Your existing functions
  async function decrypt(encryptedDataWithIV: string, key: Uint8Array): Promise<string> {
    try {
      const buffer = new Uint8Array(
        atob(encryptedDataWithIV).split('').map(char => char.charCodeAt(0))
      );
      const iv = buffer.slice(0, 12);
      const encryptedData = buffer.slice(12);

      const algorithm = { name: 'AES-GCM', iv };
      const cryptoKey = await crypto.subtle.importKey('raw', key, algorithm, false, ['decrypt']);

      const decryptedData = await crypto.subtle.decrypt(algorithm, cryptoKey, encryptedData);
      return new TextDecoder().decode(decryptedData);
    } catch (error) {
      console.warn("Decryption failed, using fallback:", error);
      return '';
    }
  }

  async function getCustomerData(cid: string) {
    try {
      const response = await fetch(`https://vnnex1njb9.execute-api.ap-south-1.amazonaws.com/test/customer/getUsingCID/${cid}`);
      const data = await response.json();
      localStorage.setItem('customerID', data.CustomerID);
      localStorage.setItem('firstName', data.FName);
      localStorage.setItem('lastName', data.LName);
      localStorage.setItem('address', data.Address);
      localStorage.setItem('phone', data.PhoneNumber);
      localStorage.setItem('phoneNumber', data.PhoneNumber);
      localStorage.setItem('email', data.Email);
    } catch (err) {
      console.error("Error fetching customer data:", err);
    }
  }

  async function getTimeZone(cid: string) {
    try {
      const res = await fetch(`https://vnnex1njb9.execute-api.ap-south-1.amazonaws.com/test/device/getAll/${cid}`);
      const data = await res.json();
      if (!data.length || data.error === 'No devices found !') {
        localStorage.setItem('TimeZone', 'PST');
      } else {
        localStorage.setItem('TimeZone', data[0]?.TimeZone || 'PST');
      }
    } catch (err) {
      console.error("Error fetching timezone:", err);
      localStorage.setItem('TimeZone', 'PST');
    }
  }

  async function handleSubmit() {
    errorMsg = '';
    if (!username.trim() || !password.trim()) {
      errorMsg = 'Please enter both username and password';
      return;
    }

    loading = true;

    try {
      const res = await fetch(`https://vnnex1njb9.execute-api.ap-south-1.amazonaws.com/test/company/getuser/${username}`);
      if (!res.ok) throw new Error('User fetch failed');

      const data = await res.json();

      if (!data || !data.Password || !data.UserName) {
        errorMsg = 'Invalid response from server';
        return;
      }

      
      const decryptedPwd = await decrypt(data.Password, key1);

      const isAuthenticated =
        data.UserName === username &&
        (decryptedPwd === password || data.Password === password);

      if (!isAuthenticated) {
        errorMsg = 'Invalid username or password';
        return;
      }

      // Set LocalStorage
      localStorage.setItem('companyID', data.CID);
      localStorage.setItem('companyName', data.CName);
      localStorage.setItem('companyLogo', data.CLogo);
      localStorage.setItem('companyAddress', data.CAddress);
      localStorage.setItem('username', data.UserName);
      localStorage.setItem('password', data.Password);
      localStorage.setItem('reportType', data.ReportType);

      window.location.href = '/employeelist';

      if (data.CLogo) {
        logoSrc = data.CLogo;
      }

      await Promise.all([
        getCustomerData(data.CID),
        getTimeZone(data.CID)
      ]);

      // window.location.href = '/employeelist';
    } catch (err) {
      console.error("Login error:", err);
      errorMsg = 'An error occurred during login';
    } finally {
      loading = false;
    }
  }
</script>

<div class="min-h-screen flex flex-col md:flex-row">
  <!-- Left side image & intro -->
  <div class="hidden md:flex xl:w-1/2 md:w-1/2 bg-[#D9E9FB] flex-col justify-center items-center xl:p-6">
    <div class="w-full flex flex-col items-center text-center">
      <img src="/icode-logo.png" alt="icode-logo" class="w-44 xl:w-94 md:w-32 mb-12 pt-42" />
      <h3 class="text-2xl xl:text-3xl md:text-2xl font-semibold text-gray-800 mb-2">Employee Time Tracking</h3>
      <p class="text-gray-900">One tap solution for simplifying and streamlining employee time logging and reporting.</p>
    </div>
  </div>

  <!-- Right side login form -->
  <div class="w-full md:w-1/2 flex justify-center items-center py-16 px-6 sm:px-8 md:px-8 md:pt-30 lg:px-20">
    <div class="w-full max-w-md bg-white rounded-xl p-6 sm:p-8 text-center">
      <h2 class="sm:text-3xl md:text-3xl text-2xl pt-4 font-semibold text-gray-800 mb-10">Login</h2>

      <div class="mb-6">
        <input
          type="text"
          bind:value={username}
          placeholder="User Name"
          class="w-full px-6 py-4 rounded-lg border-2 border-[#02066F] focus:outline-none text-center font-bold text-base sm:text-lg"
          required
        />
      </div>

      <div class="mb-4">
        <input
          type="password"
          bind:value={password}
          placeholder="Password"
          class="w-full px-6 py-4 rounded-lg border-2 border-[#02066F] focus:outline-none text-center font-bold text-base sm:text-lg"
          required
        />
      </div>

      {#if errorMsg}
        <div class="text-red-500 font-semibold mb-4">{errorMsg}</div>
      {/if}

      <div class="mb-6">
        <a href="/forgetpassword" class="text-[#02066F] font-bold">Forget password?</a>
      </div>

      <button
        on:click={handleSubmit}
        class="w-full bg-[#02066F] text-white text-lg cursor-pointer sm:text-xl py-4 rounded-lg transition duration-300 mb-4 disabled:opacity-50 disabled:cursor-not-allowed"
        disabled={loading}
      >
        {loading ? 'Logging in...' : 'Submit'}
      </button>

      <div>
        <span class="text-base sm:text-xl font-bold text-gray-800">Don't have an account?</span>
        <a href="/register" class="ml-2 text-[#02066F] text-base sm:text-xl font-bold hover:underline">Signup</a>
      </div>

      <div class="relative my-6">
        <div class="absolute inset-0 flex items-center">
          <div class="w-full border-t border-gray-300"></div>
        </div>
        <div class="relative flex justify-center">
          <span class="px-2 bg-white font-bold text-gray-800">or</span>
        </div>
      </div>

      <!-- Google Sign-In Button -->
      <div class="w-full flex justify-center mb-">
        <div 
          id="googleSignInBtn" 
          class="google-signin-btn" 
          style="width: 300px; height: 44px;"
        ></div>
      </div>

    </div>
  </div>

  {#if loading}
    <div class="fixed inset-0 flex items-center justify-center z-50"
    style="background: rgba(0, 0, 0, 0.5)">
      <div class="animate-spin w-12 h-12 border-t-4 border-b-4 border-[#02066F] rounded-full"></div>
    </div>
  {/if}
</div>

