<script lang="ts">
    import { onMount } from 'svelte';
    import { v4 as uuidv4 } from 'uuid';
    import intlTelInput from 'intl-tel-input';
    import 'intl-tel-input/build/css/intlTelInput.css';
    import { loadStripe } from '@stripe/stripe-js';

    // Constants
    const isAlpha = /^[a-zA-Z\s]+$/;
    const apiUrlBase = 'https://1wwsjsc00f.execute-api.ap-south-1.amazonaws.com/test/customer';
    const firstSignupPageapiUrlBase = 'https://1wwsjsc00f.execute-api.ap-south-1.amazonaws.com/test/company';
    const cid = uuidv4();
    const key = new Uint8Array([16, 147, 220, 113, 166, 142, 22, 93, 241, 91, 13, 252, 112, 122, 119, 95]);
    const stripePromise = loadStripe('pk_test_51OB8JlIPoM7JHRT2DlaE8KmPRFkgeSXkqf4eQZxEahu0Lbno3vHzCTH5J4rDAfw53PjdWlLteNJNzPVdahkzTb8100DA6sqAp4');

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
    let errorStreet = '';
    let errorCity = '';
    let errorState = '';
    let errorZip = '';
    let errorEmail = '';
    let totalError = '';

    // UI states
    let showOverlay = false;
    let iti: any;

    // Error map for phone validation
    const errorMap = {
        "-99": "Invalid number",
        "-2": "Invalid country code",
        "-1": "Invalid number",
        "0": "Invalid number",
        "1": "Invalid number length",
        "2": "Invalid number"
    };

    // Validation functions
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

    function validateEmail(): boolean {
        const emailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        if (email.trim() === '') {
            errorEmail = '';
            return false;
        } else if (!emailPattern.test(email)) {
            errorEmail = 'Invalid email format';
            return false;
        }
        errorEmail = '';
        return true;
    }

    function validateAddress(): boolean {
        let isValid = true;
        
        if (customerStreet.trim() === '') {
            errorStreet = 'Street is required';
            isValid = false;
        } else {
            errorStreet = '';
        }

        if (customerCity.trim() === '') {
            errorCity = 'City is required';
            isValid = false;
        } else {
            errorCity = '';
        }

        if (customerState.trim() === '') {
            errorState = 'State is required';
            isValid = false;
        } else {
            errorState = '';
        }

        if (customerZip.trim() === '') {
            errorZip = 'Zip code is required';
            isValid = false;
        } else {
            errorZip = '';
        }

        return isValid;
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

    // API functions
    async function createCheckoutSession(): Promise<void> {
        showOverlay = true;
        totalError = '';
        
        try {
            const link2 = "http://localhost:5173";
            const link = "https://tap-time.com";
            const link3 = "https://arunkavitha1982.github.io/icode";
            
            const response = await fetch(
                'https://1wwsjsc00f.execute-api.ap-south-1.amazonaws.com/test/create-checkout-session', 
                {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        "url": link2,
                        "productName": "EMS Product",
                        "amount": 2000
                    })
                }
            );

            if (!response.ok) {
                const errorDetails = await response.json();
                throw new Error(errorDetails.error);
            }

            const session = await response.json();
            
            // Initialize Stripe
            const stripe = await stripePromise;
            if (!stripe) {
                throw new Error('Stripe failed to initialize');
            }
            
            const { error } = await stripe.redirectToCheckout({ sessionId: session.id });
            if (error) {
                throw error;
            }

        } catch (error) {
            console.error('Checkout error:', error);
            totalError = error instanceof Error ? error.message : 'Payment processing failed';
        } finally {
            showOverlay = false;
        }
    }

    

    function createApiData(): void {
        const customerId = uuidv4();
        const apiUrl = `${apiUrlBase}/create`;

        const firstName = localStorage.getItem('firstName');
        const lastName = localStorage.getItem('lastName');
        const address = localStorage.getItem('address');
        const phone = localStorage.getItem('phone');
        const email = localStorage.getItem('email');

        const userData = {
            CustomerID: customerId.toString(),
            CID: cid,
            FName: firstName,
            LName: lastName,
            Address: address,
            PhoneNumber: phone,
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
            window.location.href = "/login";
        })
        .catch(error => {
            console.error('Customer creation error:', error);
        });
    }

    // Form validation
    async function validateForm(event: Event): Promise<void> {
        event.preventDefault();
        showOverlay = true;
        totalError = '';
        
        const isFirstNameValid = validateFirstName();
        const isLastNameValid = validateLastName();
        const isPhoneNumberValid = validPhoneno();
        const isAddressValid = validateAddress();
        const isEmailValid = validateEmail();

        if (isFirstNameValid && isLastNameValid && isPhoneNumberValid && isAddressValid && isEmailValid) {
            const companyStreet = localStorage.getItem('companyStreet');
            const companyCity = localStorage.getItem('companyCity');
            const companyState = localStorage.getItem('companyState');
            const companyZip = localStorage.getItem('companyZip');

            localStorage.setItem('firstName', firstName);
            localStorage.setItem('lastName', lastName);
            localStorage.setItem('address', `${companyStreet}--${companyCity}--${companyState}--${companyZip}`);
            localStorage.setItem('phone', phoneNumber);
            localStorage.setItem('email', email);

            await createCheckoutSession();
        } else {
            totalError = 'Please fix the errors';
            showOverlay = false;
        }
    }

    onMount(() => {
        // Initialize phone input
        const phoneInput = document.getElementById('phoneNumber') as HTMLInputElement;
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
         <img src="/tap-time-logo.png" alt="icode-logo" class="w-44 xl:w-94 md:w-32 md:pt-2 xl:pt-8 ">
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
        
          <form class="w-full pt-14 md:pt-12 ">
            <h2 class="text-center text-3xl xl:text-3xl md:text-2xl text-gray-800 font-semibold mb-4">Signup</h2>
        
            <!-- Horizontal line -->
            <hr class="w-full h-4 bg-gray-200 rounded-full border-4 mb-6 border-gray-200" />
  
        
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
                placeholder="Customer Address Line 1"
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
    