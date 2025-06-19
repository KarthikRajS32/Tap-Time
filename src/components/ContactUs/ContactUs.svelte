<script lang="ts">
  import { onMount } from 'svelte';
  import { v4 as uuidv4 } from 'uuid';

  let cname = '';
  let cemail = '';
  let question = '';
  let phoneNumber = '';
  
  let errorName = '';
  let errorTextarea = '';
  let errorPhone = '';
  
  let showSuccessModal = false;

  // Validate name field
  const validCName = (): boolean => {
    const isAlpha = /^[a-zA-Z\s]+$/;
    
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

  // Validate message field
  const validCQueries = (): boolean => {
    const maxLength = 500;
    const currentLength = question.length;

    if (question.trim() === '') {
      errorTextarea = '';
      return false;
    } else if (currentLength >= maxLength) {
      errorTextarea = 'Maximum character limit reached.';
      question = question.substring(0, maxLength);
      return false;
    } else {
      errorTextarea = '';
      return true;
    }
  };

  // Format phone number
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

  // Validate phone number
  const validatePhoneNumber = (): boolean => {
    const phoneRegex = /^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$/;

    if (phoneNumber.trim() === '') {
      errorPhone = '';
      return false;
    } else if (!phoneRegex.test(phoneNumber)) {
      errorPhone = 'Invalid phone number.';
      return false;
    } else {
      errorPhone = '';
      return true;
    }
  };

  // Submit form
  const handleSubmit = async (e: Event) => {
    e.preventDefault();
    
    const isNameValid = validCName();
    const isValidMessage = validCQueries();
    const isPhoneNumberValid = validatePhoneNumber();
    
    if (!cname || !cemail || !question || !phoneNumber) {
      // Basic required field validation
      return;
    }

    if (isNameValid && isValidMessage && isPhoneNumberValid) {
      await callContactUsCreateAPiData();
      showSuccessModal = true;
      
      // Reset form
      cname = '';
      cemail = '';
      question = '';
      phoneNumber = '';
      
      // Hide modal after 2 seconds
      setTimeout(() => {
        showSuccessModal = false;
      }, 2000);
    }
  };

  // API call
  const callContactUsCreateAPiData = async () => {
    const apiLink = `https://yrvi6y00u8.execute-api.us-west-2.amazonaws.com/dev/contact-us/create`;
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

    try {
      const response = await fetch(apiLink, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(userData)
      });

      if (!response.ok) {
        throw new Error(`Error: ${response.status}`);
      }

      const data = await response.json();

      if (data.error) {
        alert(data.error);
      }
    } catch (error) {
      console.error('Error submitting form:', error);
    }
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
          placeholder="Email"
          class="w-full px-4 py-3 border-2 border-[#02066F] rounded-lg font-bold focus:outline-none bg-white"
          required
        />
      </div>
      
      <!-- Message -->
      <div>
        <textarea
          id="question"
          bind:value={question}
          on:blur={validCQueries}
          placeholder="Message/Queries"
          class="w-full px-4 py-3 border-2 border-[#02066F] rounded-lg font-bold focus:outline-none bg-white min-h-[120px]"
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
      <div class="bg-white rounded-lg shadow-xl max-w-sm w-full mx-4">
        <div class="bg-blue-900 text-white py-4 px-6 rounded-t-lg text-center">
          <h5 class="text-xl font-bold">Thank You for Contacting Us!</h5>
        </div>
        <div class="p-6 text-center">
          <p class="font-bold mb-4">
            We have received your message and will get back to you shortly.
          </p>
          <div class="flex justify-center">
            <img 
              src="https://www.shutterstock.com/image-vector/blue-check-mark-icon-tick-260nw-787016416.jpg" 
              alt="Checkmark" 
              class="w-24 h-24"
            />
          </div>
        </div>
      </div>
    </div>
  {/if}
</section>