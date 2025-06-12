<script lang="ts">
      import { onMount } from 'svelte';
      import { v4 as uuidv4 } from 'uuid';
      import intlTelInput from 'intl-tel-input';
      import 'intl-tel-input/build/css/intlTelInput.css';
    
      // Regex
      const isAlpha = /^[a-zA-Z\s]+$/;
      let keyBytes: Uint8Array;

      // Form states
      let firstName = '';
      let lastName = '';
      let phoneNumber = '';
      let email = '';
    
      let customerStreet = '';
      let customerCity = '';
      let customerState = '';
      let customerZip = '';
    
      // Error messages
      let errorFirstName = '';
      let errorLastName = '';
      let errorPhone = '';
      let errorEmail = '';
      let totalError = '';
    
      // Overlay + Progress
      let showOverlay = false;
      let progress = 0;
    
      // intl-tel-input instance
      // let phoneInput: HTMLInputElement;
      let iti: any;
    
      const cid = uuidv4(); // Company ID
    
      // Validation
      function validateFirstName() {
        if (!firstName.trim() || !isAlpha.test(firstName)) {
          errorFirstName = 'Only use letters and spaces';
          return false;
        }
        errorFirstName = '';
        return true;
      }
  
      function validateLastName() {
        if (!lastName.trim() || !isAlpha.test(lastName)) {
          errorLastName = 'Only use letters and spaces';
          return false;
        }
        errorLastName = '';
        return true;
      }

      function validateEmail() {
        // Simple email regex for demonstration
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        if (!email.trim() || !emailRegex.test(email)) {
          errorEmail = 'Invalid email address';
          return false;
        }
        errorEmail = '';
        return true;
      }
  
      function formatPhone() {
        let v = phoneNumber.replace(/\D/g, '');
        if (v.length > 6) v = `(${v.slice(0, 3)}) ${v.slice(3, 6)}-${v.slice(6, 10)}`;
        else if (v.length > 3) v = `(${v.slice(0, 3)}) ${v.slice(3)}`;
        else v = v;
        phoneNumber = v;
      }
  
      // function validatePhone() {
      //   if (!iti?.isValidNumber?.()) {
      //     errorPhone = 'Invalid phone number';
      //     return false;
      //   }
      //   errorPhone = '';
      //   return true;
      // }
    
      function validatePhone() {
      const cleaned = phoneNumber.replace(/\D/g, '');
      const isValid = /^[2-9]\d{2}[2-9]\d{2}\d{4}$/.test(cleaned); // US phone format
      if (!isValid) {
        errorPhone = 'Invalid phone number';
        return false;
      }
      errorPhone = '';
      return true;
      }

      // AES-GCM encryption
      async function encrypt(data: string, key: Uint8Array): Promise<string> {
        const encoder = new TextEncoder();
        const encodedData = encoder.encode(data);
        const iv = crypto.getRandomValues(new Uint8Array(12));
        const cryptoKey = await crypto.subtle.importKey('raw', key, 'AES-GCM', false, ['encrypt']);
        const encrypted = await crypto.subtle.encrypt({ name: 'AES-GCM', iv }, cryptoKey, encodedData);
        const combined = new Uint8Array(iv.length + encrypted.byteLength);
        combined.set(iv);
        combined.set(new Uint8Array(encrypted), iv.length);
        return btoa(String.fromCharCode(...combined));
      }
    
      // Get and encrypt password from localStorage
      async function checkPassword(): Promise<string> {
        const pwd = localStorage.getItem('password') || '';
        try {
          return await encrypt(pwd, keyBytes);
        } catch (error) {
          console.error('Encryption failed', error);
          return '';
        }
      }
    
      // Stripe checkout
      async function createCheckoutSession() {
        try {
          const resp = await fetch('https://yrvi6y00u8.execute-api.us-west-2.amazonaws.com/dev/create-checkout-session', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
              url: location.origin,
              productName: 'EMS Product',
              amount: 2000
            })
          });
    
          if (!resp.ok) throw await resp.json();
    
          const session = await resp.json();
          const stripe = (window as any).Stripe('pk_test_...');
          await stripe.redirectToCheckout({ sessionId: session.id });
        } catch (error) {
          console.error('Stripe checkout failed', error);
        }
      }
    
      // Create company
      async function createCompany() {
        const url = 'https://yrvi6y00u8.execute-api.us-west-2.amazonaws.com/dev/company/create';
        const companyData = {
          CID: cid,
          CName: localStorage.getItem('companyName'),
          CLogo: localStorage.getItem('companyLogo'),
          CAddress: `${localStorage.getItem('companyStreet')} -- ${localStorage.getItem('companyCity')} -- ${localStorage.getItem('companyState')} -- ${localStorage.getItem('companyZip')}`,
          UserName: localStorage.getItem('username'),
          Password: await checkPassword(),
          ReportType: 'Weekly',
          LastModifiedBy: 'Admin'
        };
    
        await fetch(url, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(companyData)
        });
      }
    
      // Create customer
      async function createCustomer() {
        const url = 'https://yrvi6y00u8.execute-api.us-west-2.amazonaws.com/dev/customer/create';
        const customerData = {
          CustomerID: uuidv4(),
          CID: cid,
          FName: firstName,
          LName: lastName,
          Address: `${customerStreet} -- ${customerCity} -- ${customerState} -- ${customerZip}`,
          PhoneNumber: phoneNumber,
          Email: email,
          IsActive: true,
          LastModifiedBy: 'Admin'
        };
    
        await fetch(url, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(customerData)
        });
      }
    
      // Final form validation and flow
      async function validateForm(e: Event) {
        e.preventDefault();
        showOverlay = true;
        progress = 0;
        totalError = '';
    
        const isFNValid = validateFirstName();
        const isLNValid = validateLastName();
        formatPhone();
        const isPhoneValid = validatePhone();
    
        // Basic presence check
        const allFilled = firstName && lastName && phoneNumber && email &&
          customerStreet && customerCity && customerState && customerZip;
    
        if (!allFilled) {
          totalError = 'Please fill all required fields.';
          showOverlay = false;
          return;
        }
    
        if (isFNValid && isLNValid && isPhoneValid) {
          progress = 50;
    
          // Store in localStorage
          localStorage.setItem('firstName', firstName);
          localStorage.setItem('lastName', lastName);
          localStorage.setItem('customerStreet', customerStreet);
          localStorage.setItem('customerCity', customerCity);
          localStorage.setItem('customerState', customerState);
          localStorage.setItem('customerZip', customerZip);
          localStorage.setItem('phone', phoneNumber);
          localStorage.setItem('email', email);
    
          await createCompany();
          progress = 80;
    
          await createCustomer();
          progress = 100;
    
          // Success visual
          const progressBar = document.getElementById('progress-bar');
          progressBar?.classList.add('filled');
    
          showOverlay = false;
          await createCheckoutSession();
        } else {
          totalError = 'Please fix the validation errors.';
          showOverlay = false;
        }
      }
    
      // Initialize phone input on mount
      onMount(() => {
  // Safe to access localStorage here (client-side only)
  keyBytes = new Uint8Array([16, 147, 220, 113, 166, 142, 22, 93, 241, 91, 13, 252, 112, 122, 119, 95]);
  localStorage.setItem('key', JSON.stringify(Array.from(keyBytes)));

  // Initialize intl-tel-input
  // iti = intlTelInput(phoneInput, {
  //   initialCountry: 'auto',
  //   geoIpLookup: (callback: (countryCode: string) => void) => {
  //     fetch('https://ipapi.co/json')
  //       .then(response => response.json())
  //       .then(data => callback(data.country_code))
  //       .catch(() => callback('us'));
  //   },
  //   utilsScript: 'https://cdn.jsdelivr.net/npm/intl-tel-input@17.0.19/build/js/utils.js'
  // } as any);
});
    </script>
      

    
    <div class="flex flex-col md:flex-row min-h-screen">
      <!-- Left Section -->
      <div class="hidden md:flex md:w-1/2 bg-blue-100 flex-col justify-center items-center p-5  ">
        <!-- <img src="/icode-logo.png" alt="icode-logo" class="w-18 mb-4" /> -->
         <img src="/icode-logo.png" alt="icode-logo" class="w-44 xl:w-94 md:w-32 md:pt-2 xl:pt-8 ">
        <div>
         <h3 class="text-center text-3xl xl:text-3xl md:text-2xl text-gray-800 font-semibold xl:pt-20 md:pt-18">Join Us Today</h3>
        <p class="text-center text-gray-700 mt-2 ">
          Create an account to unlock seamless time tracking and boost your team's efficiency.
        </p>
      </div>
      </div>
    
      <!-- Right Section -->
      
        <div class="w-full md:w-1/2 xl:w-1/2 flex justify-center items-center p-6 pt-10 xl:pt-16 md:pt-16 ">
          {#if showOverlay}
            <div class="fixed inset-0 bg-black bg-opacity-50 flex justify-center items-center z-50">
              <div class="border-4 border-t-4 border-[#02066F] rounded-full w-10 h-10 animate-spin"></div>
            </div>
          {/if}
        
          <form class="w-full pt-16 md:pt-24 ">
            <h2 class="text-center text-3xl xl:text-3xl md:text-2xl text-gray-800 font-semibold mb-4">Signup</h2>
        
            <!-- Horizontal line -->
            <hr class="w-full rounded-full border-4 mb-6" style="border-color:  rgb(234,234,234);" />
  
        
            {#if totalError}
              <p class="text-red-600 text-sm mb-2">{totalError}</p>
            {/if}
        
            <!-- Company Name -->
            <div class="border-2 border-[#02066F] rounded-lg  mb-4">
             
            <input
            type="text"
            bind:value={firstName}
            on:blur={validateFirstName}
            placeholder="First Name"
            class="w-full outline-none font-bold p-3 xl:p-5 md:p-3"
            required
            />
            </div>
            {#if errorFirstName}
            <p class="text-red-600 text-sm mb-2">{errorFirstName}</p>
            {/if}
           
            <div class="border-2 border-[#02066F] rounded-lg  mb-4">
            <input
            type="text"
            bind:value={lastName}
            on:blur={validateLastName}
            placeholder="Last Name"
            class="w-full outline-none font-bold p-3 xl:p-5 md:p-3"
            required
            />
            </div>
            {#if errorLastName}
            <p class="text-red-600 text-sm mb-2">{errorLastName}</p>
            {/if}

        
            <!-- Customer Address -->
            <div class="border-2 border-[#02066F] rounded-lg p-2 xl:p-4 md:p-4 mb-4">
              <p class="text-center text-[#02066F] font-bold mb-4">Customer Address</p>
              <div class="flex flex-col md:flex-row gap-4">
                
                <input
                type="text"
                bind:value={customerStreet}
                placeholder="Customer Street"
                class="w-full border-2 border-[#02066F] rounded-lg p-2 xl:p-6 md:p-3 font-bold focus:outline-none"
                required
                />
                
                <input
                type="text"
                bind:value={customerCity}
                placeholder="Customer City"
                class="w-full border-2 border-[#02066F] rounded-lg p-2 xl:p-6 md:p-3 font-bold focus:outline-none"
                required
                />
              </div>
              
              <div class="flex flex-col md:flex-row gap-4 mt-4 mb-4">
                
                <input
                type="text"
                bind:value={customerState}
                placeholder="Customer State"
                class="w-full border-2 border-[#02066F] rounded-lg p-2 xl:p-6 md:p-3 font-bold focus:outline-none"
                required
                />
                
                <input
                type="text"
                bind:value={customerZip}
                placeholder="Customer Zip"
                class="w-full border-2 border-[#02066F] rounded-lg p-2 xl:p-6 md:p-3 font-bold focus:outline-none"
                required
                />
              </div>
            </div>
        


            <div class="border-2 border-[#02066F] rounded-lg mb-4">
                <input
                  type="tel"
                  bind:value={phoneNumber}
                  on:input={formatPhone}
                  on:blur={validatePhone}
                  placeholder="Phone Number"
                  class="w-full outline-none font-bold p-2 xl:p-5 md:p-3"
                  required
                />
              </div>
              {#if errorPhone}
                <p class="text-red-600 text-sm mb-2">{errorPhone}</p>
              {/if}
        


            <div class="border-2 border-[#02066F] rounded-lg mb-4">
                <input
                  type="email"
                  bind:value={email}
                  on:blur={validateEmail}
                  placeholder="Email"
                  class="w-full outline-none font-bold p-2 xl:p-5 md:p-3"
                  required
                />
              </div>
              {#if errorEmail}
                <p class="text-red-600 text-sm mb-2">{errorEmail}</p>
              {/if}
        
    
            <button
            type="submit"
            class="w-full bg-[#02066F] text-white py-3 rounded-lg text-lg mt-4 cursor-pointer"
            on:click={validateForm}
            >
            Pay
            </button>
            </form>
                {#if showOverlay}
                <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
                <div class="text-white text-xl font-semibold">Processing...</div>
                </div>
            {/if}
        </div>
        
    </div>
    