<script lang="ts">
  import { onMount } from 'svelte';
  import { v4 as uuidv4 } from 'uuid';
  import { fade } from 'svelte/transition';

  // Form fields
  let cname = '';
  let cemail = '';
  let question = '';
  let phoneNumber = '';
  
  // Error messages
  let errorName = '';
  let errorEmail = '';
  let errorTextarea = '';
  let errorPhone = '';
  
  // Modal states
  let showSuccessModal = false;
  
  // Regular expressions
  const isAlpha = /^[a-zA-Z\s]+$/;
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  const phoneRegex = /^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$/;
  const maxMessageLength = 500;


  // Field validations
  const validCName = (): boolean => {
    if (cname.trim() === '') {
      errorName = '';
      return false;
    } else if (!isAlpha.test(cname)) {
      errorName = 'Only use letters, don\'t use digits';
      return false;
    } else {
      errorName = '';
      return true;
    }
  };

  const validCEmail = (): boolean => {
    if (cemail.trim() === '') {
      errorEmail = '';
      return false;
    } else if (!emailRegex.test(cemail)) {
      errorEmail = 'Please enter a valid email address';
      return false;
    } else {
      errorEmail = '';
      return true;
    }
  };

  const validCQueries = (): boolean => {
    if (question.trim() === '') {
      errorTextarea = '';
      return false;
    } else if (question.length >= maxMessageLength) {
      errorTextarea = `Maximum ${maxMessageLength} characters allowed`;
      question = question.substring(0, maxMessageLength);
      return false;
    } else {
      errorTextarea = '';
      return true;
    }
  };

  const formatPhoneNumber = () => {
    let value = phoneNumber.replace(/\D/g, '');
    
    if (value.length > 3 && value.length <= 6) {
      value = `(${value.slice(0, 3)}) ${value.slice(3)}`;
    } else if (value.length > 6) {
      value = `(${value.slice(0, 3)}) ${value.slice(3, 6)}-${value.slice(6, 10)}`;
    } else if (value.length > 3) {
      value = `(${value.slice(0, 3)}) ${value.slice(3)}`;
    }
    
    phoneNumber = value;
  };

  const validatePhoneNumber = (): boolean => {
    if (phoneNumber.trim() === '') {
      errorPhone = '';
      return false;
    } else if (!phoneRegex.test(phoneNumber)) {
      errorPhone = 'Please use format: (123) 456-7890';
      return false;
    } else {
      errorPhone = '';
      return true;
    }
  };

  // Submit form
  const handleSubmit = async (e: Event) => {
    e.preventDefault();
    
    // Validate all fields
    const isNameValid = validCName();
    const isEmailValid = validCEmail();
    const isValidMessage = validCQueries();
    const isPhoneNumberValid = validatePhoneNumber();
    
    // Check required fields (name, email, question)
    const isRequiredFieldsValid = cname.trim() !== '' && cemail.trim() !== '' && question.trim() !== '';
    
    if (isNameValid && isEmailValid && isValidMessage && isPhoneNumberValid && isRequiredFieldsValid) {
      try {
        await callContactUsCreateAPiData();
        showSuccessModal = true;
        
        // Hide modal after 2 seconds
        setTimeout(() => {
          showSuccessModal = false;
        }, 2000);
      } catch (error) {
        console.error('Form submission failed:', error);
      }
    } else {
      // Trigger validation messages for required fields
      if (cname.trim() === '') errorName = 'Name is required';
      if (cemail.trim() === '') errorEmail = 'Email is required';
      if (question.trim() === '') errorTextarea = 'Message is required';
    }
  };

  // API call
  const callContactUsCreateAPiData = async (): Promise<void> => {
    const apiLink = `https://vnnex1njb9.execute-api.ap-south-1.amazonaws.com/test/contact-us/create`;
    const requestID = uuidv4();
    const cid = localStorage.getItem('companyID') || '';

    const userData = {
      RequestID: requestID,
      CID: cid,
      Name: cname,
      RequestorEmail: cemail,
      ConcernsQuestions: question,
      PhoneNumber: phoneNumber,
      Status: "pending",
      LastModifiedBy: 'Admin'
    };

    const response = await fetch(apiLink, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(userData)
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();

    if (data.error) {
      throw new Error(data.error);
    }

    // Reset form only after successful submission
    cname = '';
    cemail = '';
    question = '';
    phoneNumber = '';
  };
</script>

<section class="min-h-full bg-gray-100 px-6 pt-28 pb-12">
  <div class="max-w-md mx-auto bg-gray-50 rounded-lg shadow-xl  overflow-hidden p-6 px-10">
    <h3 class="text-3xl font-bold text-center text-gray-800 mb-6">Contact Us</h3>
    
    <form id="emp_form" class="space-y-4" on:submit={handleSubmit}>
      <!-- Name -->
      <div>
        <input
          type="text"
          id="cname"
          bind:value={cname}
          on:blur={validCName}
          placeholder="Name"
          class="w-full px-4 py-3 border-2 border-[#02066F] rounded-lg font-bold focus:outline-none bg-white"
          required
        />
        {#if errorName}
          <p class="text-red-500 text-sm mt-1">{errorName}</p>
        {/if}
      </div>
      
      <!-- Email -->
      <div>
        <input
          type="email"
          id="cemail"
          bind:value={cemail}
          on:blur={validCEmail}
          placeholder="Email"
          class="w-full px-4 py-3 border-2 border-[#02066F] rounded-lg font-bold focus:outline-none bg-white"
          required
        />
        {#if errorEmail}
          <p class="text-red-500 text-sm mt-1">{errorEmail}</p>
        {/if}
      </div>
      
      <!-- Message -->
      <div>
        <textarea
          id="question"
          bind:value={question}
          on:blur={validCQueries}
          placeholder="Message/Queries"
          class="w-full px-4 py-3 border-2 border-[#02066F] rounded-lg font-bold focus:outline-none bg-white min-h-[90px]"
          required
        ></textarea>
        {#if errorTextarea}
          <p class="text-red-500 text-sm mt-1">{errorTextarea}</p>
        {/if}
      </div>
      
      <!-- Phone Number -->
      <div>
        <input
          type="tel"
          id="phoneNumber"
          bind:value={phoneNumber}
          on:input={formatPhoneNumber}
          on:blur={validatePhoneNumber}
          placeholder="Phone Number"
          class="w-full px-4 py-3 border-2 border-[#02066F] rounded-lg font-bold focus:outline-none bg-white"
          required
        />
        {#if errorPhone}
          <p class="text-red-500 text-sm mt-1">{errorPhone}</p>
        {/if}
      </div>
      
      <!-- Email Link -->
      <div class="flex flex-col md:flex md:flex-row items-center justify-center text-sm sm:text-base gap-2">
        <span class="font-bold text-gray-800 mr-2">Write to us at : </span>
        <a 
          href="mailto:contact@tap-time.com" 
          target="_blank"
          class="text-blue-600 hover:underline"
        >
          contact@tap-time.com
        </a>
      </div>
      
      <!-- Submit Button -->
      <div class=" pb-4 text-center">
        <button
          type="submit"
          class="w-auto bg-[#02066F] text-white py-3 px-8 rounded-md font-bold cursor-pointer transition duration-200"
        >
          Submit
        </button>
      </div>
    </form>
  </div>
  
  <!-- Success Modal -->
  {#if showSuccessModal}
    <div class="fixed inset-0 flex items-center justify-center z-50"
    style="background: rgba(0, 0, 0, 0.5)">
    <div class="bg-white rounded-sm shadow-xl w-full max-w-sm sm:max-w-lg mx-auto my-6 sm:my-12">
      <!-- Header -->
      <div class="bg-[#02066F] text-white py-4 px-4 sm:px-6 rounded-t-sm text-center">
        <h5 class="text-lg sm:text-xl font-semibold">
          Thank You for Contacting Us!
        </h5>
      </div>
    
      <!-- Body -->
      <div class="p-4 sm:p-6 text-center">
        <p class="font-bold mb-4 text-base sm:text-xl">
          We have received your message and will get back to you shortly.
        </p>
    
        <!-- Image -->
        <div class="flex justify-center">
          <img 
            src="https://www.shutterstock.com/image-vector/blue-check-mark-icon-tick-260nw-787016416.jpg" 
            alt="Checkmark" 
            class="w-20 h-20 sm:w-24 sm:h-24 object-contain"
          />
        </div>
      </div>
    </div>
    </div>
  {/if}
</section>