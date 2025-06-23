<script lang="ts">
  import { onMount } from 'svelte';
  import { v4 as uuidv4 } from 'uuid';
  import intlTelInput from 'intl-tel-input';
  import 'intl-tel-input/build/css/intlTelInput.css';

  // Constants
  const isAlpha = /^[a-zA-Z\s]+$/;
  const apiUrlBase = 'https://yrvi6y00u8.execute-api.us-west-2.amazonaws.com/dev/customer';
  const firstSignupPageapiUrlBase = 'https://yrvi6y00u8.execute-api.us-west-2.amazonaws.com/dev/company';
  const cid = uuidv4();
  const key = new Uint8Array([16, 147, 220, 113, 166, 142, 22, 93, 241, 91, 13, 252, 112, 122, 119, 95]);

  // Form fields
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

  // UI states
  let showOverlay = false;
  let iti: any;
  // Validation functions (exact match to your HTML logic)
  function validateFirstName(): boolean {
      if (firstName.trim() === '') {
          errorFirstName = '';
          return false;
      } else if (!isAlpha.test(firstName)) {
          errorFirstName = 'Only use letters and spaces';
          return false;
      }
      errorFirstName = '';
      return true;
  }

  function validateEmail(): boolean {
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      if (email.trim() === '') {
          errorEmail = '';
          return false;
      } else if (!emailRegex.test(email)) {
          errorEmail = 'Invalid email address.';
          return false;
      }
      errorEmail = '';
      return true;
  }
  

  function validateLastName(): boolean {
      if (lastName.trim() === '') {
          errorLastName = '';
          return false;
      } else if (!isAlpha.test(lastName)) {
          errorLastName = 'Only use letters and spaces';
          return false;
      }
      errorLastName = '';
      return true;
  }

  function validPhoneno(): boolean {
      const phoneRegex = /^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$/;
      
      if (phoneNumber === "") {
          errorPhone = '';
          return false;
      } else if (!phoneRegex.test(phoneNumber)) {
          errorPhone = 'Invalid phone number.';
          return false;
      } else {
          errorPhone = '';
          return true;
      }
  }

  function formatPhoneNumber() {
      let value = phoneNumber.replace(/\D/g, '');
      if (value.length > 3 && value.length <= 6) {
          value = `(${value.slice(0, 3)}) ${value.slice(3)}`;
      } else if (value.length > 6) {
          value = `(${value.slice(0, 3)}) ${value.slice(3, 6)}-${value.slice(6, 10)}`;
      } else if (value.length > 3) {
          value = `(${value.slice(0, 3)}) ${value.slice(3)}`;
      }
      phoneNumber = value;
  }

  // Encryption (exact match to your HTML logic)
  async function encrypt(data: string, key: Uint8Array): Promise<string> {
      const dataBuffer = new TextEncoder().encode(data);
      const algorithm = { name: 'AES-GCM', iv: generateRandomBytes(12) };
      
      const importedKey = await window.crypto.subtle.importKey(
          'raw', key, algorithm, false, ['encrypt']
      );

      const encryptedData = await window.crypto.subtle.encrypt(
          algorithm, importedKey, dataBuffer
      );

      const iv = algorithm.iv;
      const encryptedDataWithIV = new Uint8Array(iv.byteLength + encryptedData.byteLength);
      encryptedDataWithIV.set(iv);
      encryptedDataWithIV.set(new Uint8Array(encryptedData), iv.byteLength);

      return btoa(String.fromCharCode(...new Uint8Array(encryptedDataWithIV)));
  }

  function generateRandomBytes(length: number): Uint8Array {
      const randomValues = new Uint8Array(length);
      window.crypto.getRandomValues(randomValues);
      return randomValues;
  }

  async function checkPassword(): Promise<string> {
      const password = localStorage.getItem('password') || '';
      try {
          const encryptedPassword = await encrypt(password, key);
          return encryptedPassword.toString();
      } catch (error) {
          console.error('Encryption error:', error);
          return '';
      }
  }

  // API functions (exact match to your HTML logic)
//   async function createCheckoutSession(): Promise<void> {
//     showOverlay = true;
//     totalError = '';
    
//     try {
//         // Get all required data from localStorage
//         const email = localStorage.getItem('email') || '';
//         const firstName = localStorage.getItem('firstName') || '';
//         const lastName = localStorage.getItem('lastName') || '';
//         const companyName = localStorage.getItem('companyName') || '';

//         // Validate essential data
//         if (!email || !firstName || !lastName) {
//             throw new Error('Customer information is incomplete');
//         }

//         // Prepare customer metadata
//         const metadata = {
//             firstName,
//             lastName,
//             email,
//             company: companyName,
//             customerId: localStorage.getItem('customerID') || '',
//             timestamp: new Date().toISOString()
//         };

//         // Create the request body
//         const requestBody = {
//             success_url: `${window.location.origin}/success`,
//             cancel_url: `${window.location.origin}/cancel`,
//             line_items: [{
//                 price_data: {
//                     currency: 'usd',
//                     product_data: {
//                         name: 'EMS Product',
//                     },
//                     unit_amount: 2000, // $20.00
//                 },
//                 quantity: 1,
//             }],
//             customer_email: email,
//             metadata,
//             mode: 'payment' // Required for one-time payments
//         };

//         // Make the API call
//         const response = await fetch(
//             'https://yrvi6y00u8.execute-api.us-west-2.amazonaws.com/dev/create-checkout-session', 
//             {
//                 method: 'POST',
//                 headers: { 
//                     'Content-Type': 'application/json',
//                     // Add any required authentication headers here
//                 },
//                 body: JSON.stringify(requestBody)
//             }
//         );

//         // Handle response errors
//         if (!response.ok) {
//             const errorData = await response.json().catch(() => ({}));
//             throw new Error(errorData.message || `Payment failed with status ${response.status}`);
//         }

//         // Get the session data
//         const session = await response.json();

//         // Initialize Stripe
//         if (!(window as any).Stripe) {
//             throw new Error('Stripe.js library not loaded');
//         }

//         const stripe = (window as any).Stripe('pk_test_51OB8JlIPoM7JHRT2DlaE8KmPRFkgeSXkqf4eQZxEahu0Lbno3vHzCTH5J4rDAfw53PjdWlLteNJNzPVdahkzTb8100DA6sqAp4');

//         // Redirect to Stripe Checkout
//         const { error } = await stripe.redirectToCheckout({
//             sessionId: session.id
//         });

//         if (error) {
//             throw error;
//         }

//     } catch (error) {
//         console.error('Checkout error:', error);
//         totalError = error instanceof Error 
//             ? error.message 
//             : 'Payment processing failed. Please try again.';
        
//         // Optional: Send error to analytics
//         // trackError('checkout_error', error);
//     } finally {
//         showOverlay = false;
//     }
// }
async function createCheckoutSession(): Promise<void> {
    showOverlay = true;
    totalError = '';
    
    try {
        // 1. Validate and prepare data
        const email = localStorage.getItem('email');
        const firstName = localStorage.getItem('firstName');
        const lastName = localStorage.getItem('lastName');
        
        if (!email || !firstName || !lastName) {
            throw new Error('Please complete all required customer information');
        }

        // 2. Create the request body
        const requestBody = {
            success_url: `${window.location.origin}/success`,
            cancel_url: `${window.location.origin}/cancel`,
            line_items: [{
                price_data: {
                    currency: 'usd',
                    product_data: { name: 'EMS Product' },
                    unit_amount: 2000,
                },
                quantity: 1,
            }],
            customer_email: email,
            metadata: {
                firstName,
                lastName,
                email,
                company: localStorage.getItem('companyName') || '',
                customerId: localStorage.getItem('customerID') || '',
                timestamp: new Date().toISOString()
            },
            mode: 'payment'
        };

        console.log('Request payload:', requestBody);

        // 3. Make API call with error handling
        const response = await fetch(
            'https://yrvi6y00u8.execute-api.us-west-2.amazonaws.com/dev/create-checkout-session', 
            {
                method: 'POST',
                headers: { 
                    'Content-Type': 'application/json',
                    
                },
                body: JSON.stringify(requestBody)
            }
        );

        // 4. Handle response
        if (!response.ok) {
            let errorDetails;
            try {
                errorDetails = await response.json();
            } catch {
                errorDetails = await response.text();
            }
            
            console.error('Backend Error Details:', {
                status: response.status,
                headers: Object.fromEntries(response.headers.entries()),
                errorDetails
            });
            
            throw new Error(
                errorDetails?.message || 
                `Payment service failed (HTTP ${response.status})`
            );
        }

        const session = await response.json();
        
        if (!session?.id) {
            throw new Error('Invalid response from payment service');
        }

        // 5. Initialize Stripe
        //@ts-ignore
        if (typeof window.Stripe === 'undefined') {
            throw new Error('Payment system not loaded. Please refresh the page.');
        }
        //@ts-ignore
        const stripe = window.Stripe('pk_test_51OB8JlIPoM7JHRT2DlaE8KmPRFkgeSXkqf4eQZxEahu0Lbno3vHzCTH5J4rDAfw53PjdWlLteNJNzPVdahkzTb8100DA6sqAp4');
        
        const { error } = await stripe.redirectToCheckout({
            sessionId: session.id
        });

        if (error) throw error;

    } catch (error) {
        console.error('Payment Error:', error);
        totalError = (typeof error === 'object' && error !== null && 'message' in error)
            ? (error as { message: string }).message
            : 'We encountered an error processing your payment.';
        
        // For debugging - show more details in development
        if (import.meta.env.DEV) {
            if (typeof error === 'object' && error !== null && 'message' in error) {
                totalError += ` (${(error as { message: string }).message})`;
            }
        }
    } finally {
        showOverlay = false;
    }
}
  async function craeteFirstPageSignupAPiData(): Promise<void> {
      const firstSignupPageapiUrl = `${firstSignupPageapiUrlBase}/create`;
      const cname = localStorage.getItem('companyName');
      const clogo = localStorage.getItem('companyLogo');
      const companyStreet = localStorage.getItem('companyStreet');
      const companyCity = localStorage.getItem('companyCity');
      const companyState = localStorage.getItem('companyState');
      const companyZip = localStorage.getItem('companyZip');
      const username = localStorage.getItem('username');
      const passwordEncrypted = await checkPassword();

      const userData = {
          CID: cid,
          CName: cname,
          CLogo: clogo,
          CAddress: `
              ${companyStreet} -- 
              ${companyCity} -- 
              ${companyState} -- 
              ${companyZip} -- 
          `,
          UserName: username,
          Password: passwordEncrypted,
          ReportType: "Weekly",
          LastModifiedBy: 'Admin'
      };

      try {
          const response = await fetch(firstSignupPageapiUrl, {
              method: 'POST',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify(userData)
          });

          if (!response.ok) {
              throw new Error(`Error: ${response.status}`);
          }

          const data = await response.json();
          if (!data.error) {
              createApiData();
          } else {
              setTimeout(() => {
                  window.location.href = "singup.html";
                  showOverlay = true;
              }, 100);
          }
      } catch (error) {
          console.error('API error:', error);
      }
  }

  function createApiData(): void {
      const customerId = uuidv4();
      const apiUrl = `${apiUrlBase}/create`;

      localStorage.setItem('firstName', firstName);
      localStorage.setItem('lastName', lastName);
      localStorage.setItem('customerStreet', customerStreet);
      localStorage.setItem('customerCity', customerCity);
      localStorage.setItem('customerState', customerState);
      localStorage.setItem('customerZip', customerZip);
      localStorage.setItem('phone', phoneNumber);
      localStorage.setItem('email', email);
      localStorage.setItem('customerID', customerId);

      const userData = {
          CustomerID: customerId.toString(),
          CID: cid,
          FName: firstName,
          LName: lastName,
          Address: `
              ${customerStreet} -- 
              ${customerCity} -- 
              ${customerState} -- 
              ${customerZip} -- 
          `,
          PhoneNumber: phoneNumber,
          Email: email,
          IsActive: true,
          LastModifiedBy: 'Admin'
      };

      fetch(apiUrl, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(userData)
      })
      .then(response => {
          if (!response.ok) {
              throw new Error(`Error: ${response.status}`);
          }
          return response.json();
      })
      .then(data => {
          window.location.href = "login.html";
      })
      .catch(error => {
          console.error('Customer creation error:', error);
      });
  }

  // Form validation (exact match to your HTML logic)
  async function validateForm(event: Event): Promise<void> {
      event.preventDefault();
      showOverlay = true;
      
      const isFirstNameValid = validateFirstName();
      const isLastNameValid = validateLastName();
      const isPhoneNumberValid = validPhoneno();
      
      // Check required fields
      let isRequiredFieldsValid = true;
      const requiredFields = [
          { value: customerStreet, error: '' },
          { value: customerCity, error: '' },
          { value: customerState, error: '' },
          { value: customerZip, error: '' }
      ];

      requiredFields.forEach(field => {
          if (!field.value.trim()) {
              isRequiredFieldsValid = false;
          }
      });

      if (isFirstNameValid && isLastNameValid && isPhoneNumberValid && isRequiredFieldsValid) {
          // Get company address from localStorage
          const companyStreet = localStorage.getItem('companyStreet');
          const companyCity = localStorage.getItem('companyCity');
          const companyState = localStorage.getItem('companyState');
          const companyZip = localStorage.getItem('companyZip');

          // Save to localStorage
          localStorage.setItem('firstName', firstName);
          localStorage.setItem('lastName', lastName);
          localStorage.setItem('address', `${companyStreet}--${companyCity}--${companyState}--${companyZip}`);
          localStorage.setItem('phone', phoneNumber);
          localStorage.setItem('email', email);

          // COMPANY API CALL
          showOverlay = false;
          const passwordEncrypted = await checkPassword();
          localStorage.setItem("passwordEncrypted", passwordEncrypted);
          await createCheckoutSession();
      } else {
          totalError = 'Please fix the errors';
          showOverlay = false;
      }
  }

  onMount(() => {
      // Initialize phone input
      const phoneInput = document.getElementById('phoneNumber') as HTMLInputElement | null;
      if (phoneInput) {
          iti = intlTelInput(phoneInput, {
              initialCountry: 'us',
              utilsScript: 'https://cdn.jsdelivr.net/npm/intl-tel-input@18.2.1/build/js/utils.js'
          } as any);
      }

      // Store encryption key
      localStorage.setItem('key', JSON.stringify(Array.from(key)));
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
            <div class="fixed inset-0  flex justify-center items-center z-50"
            style="background: rgba(0, 0, 0, 0.5)">
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
                  on:input={formatPhoneNumber}
                  on:blur={validPhoneno}
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
                <div class="fixed inset-0 flex items-center justify-center z-50"
                style="background: rgba(0, 0, 0, 0.5)">
                <div class="text-white text-xl font-semibold">Processing...</div>
                </div>
            {/if}
        </div>
        
    </div>
    