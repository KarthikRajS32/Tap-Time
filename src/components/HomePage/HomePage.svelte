<script lang="ts">

  let phone = "";
  let zip = "";
  let isValid = true;
  let showOverlay = false;
  let show = false;
 

 // Refs to access form input values
  let firstNameInput: HTMLInputElement;
  let lastNameInput: HTMLInputElement;
  let emailInput: HTMLInputElement;
  let subjectInput: HTMLInputElement;
  let phoneInput: HTMLInputElement;
  let streetInput: HTMLInputElement;
  let cityInput: HTMLInputElement;
  let stateInput: HTMLInputElement;
  let zipInput: HTMLInputElement;
  let messageInput: HTMLTextAreaElement;

  // @ts-ignore
  // const validateForm = (event) => {
  //   const form = event.target;

  //   if (form.checkValidity()) {
  //     // Valid form — show overlay
  //     showOverlay = true;

  //     setTimeout(() => {
  //       showOverlay = false;
  //       alert("Form submitted!");
  //       form.reset();
  //       // You can manually submit the form here if needed: form.submit();
  //     }, 2000);
  //   } else {
  //     // Invalid form — show default browser error messages
  //     form.reportValidity();
  //   }
  // };

  const validateForm = async (event: Event) => {
    event.preventDefault();
    const form = event.target as HTMLFormElement;

    if (form.checkValidity()) {
      showOverlay = true;

      const userData = {
        FirstName: firstNameInput.value,
        LastName: lastNameInput.value,
        Email: emailInput.value,
        WhatsappNumber: null,
        Subject: subjectInput.value,
        PhoneNumber: phoneInput.value,
        Address: `${streetInput.value}--${cityInput.value}--${stateInput.value}--${zipInput.value}`,
        Message: messageInput.value,
        LastModifiedBy: "Admin",
      };

      try {
        const apiLink = `https://yrvi6y00u8.execute-api.us-west-2.amazonaws.com/dev/web_contact_us/create`;

        const response = await fetch(apiLink, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(userData),
        });

        if (!response.ok) {
          throw new Error(`Error: ${response.status}`);
        }

        const data = await response.json();
        if (!data.error) {
          show = true;
          form.reset();
         
        }
      } catch (error) {
        form.reset();
        alert("Something went wrong. Please try again.");
        console.error(error);
      } finally {
        showOverlay = false;
      }
    } else {
      form.reportValidity();
    }
  };


// @ts-ignore
const formatPhoneNumber = (event) => {
  let value = event.target.value.replace(/\D/g, '').slice(0, 10);

  if (value.length > 6) {
    phone = `${value.slice(0, 3)}-${value.slice(3, 6)}-${value.slice(6)}`;
  } else if (value.length > 3) {
    phone = `${value.slice(0, 3)}-${value.slice(3)}`;
  } else {
    phone = value;
  }
};

  const features = [
    {
      icon: 'Mask group.png',
      title: 'Facial Recognition',
      desc: 'Snap photo to log hours instantly.',
    },
    {
      icon: 'Mask group-1.png',
      title: 'Clock In/Out',
      desc: 'Seamless one-tap login and logout solution with employee identifications.',
    },
    {
      icon: 'Mask group-2.png',
      title: 'Timesheet Reports',
      desc: 'Provides employee time reports at your preferred frequency.',
    },
    {
      icon: 'Mask group-3.png',
      title: 'Admin Dashboard',
      desc: 'Employee onboarding system for Admins.',
    },
    {
      icon: 'Mask group-4.png',
      title: 'Export Options',
      desc: 'Delivers time reports in multiple formats like CSV and PDF.',
    },
    {
      icon: 'Mask group-5.png',
      title: 'Validation',
      desc: 'Admin features to update time entries.',
    }
  ];

  function validateZip() {
    const regex = /^\d{5}(-\d{4})?$/;
    isValid = regex.test(zip);
  }

  
  let onClose = () => {
    show = false;
  };

  function handleBackgroundClick(event: MouseEvent) {
    const modalContent = document.getElementById('modal-content');
    if (modalContent && !modalContent.contains(event.target as Node)) {
      onClose();
    }
  }
</script>


<!-- Loading Overlay -->
<!-- {#if showOverlay}
<div class="fixed inset-0 bg-black bg-opacity-50 z-50 flex items-center justify-center p-10">
  <div class="w-12 h-12 border-4 border-white border-t-blue-600 rounded-full animate-spin"></div>
</div>
{/if} -->

<!-- Header -->
<section class="max-w-7xl mx-auto px-4 pt-24 text-center">
<h2 class="text-3xl font-bold text-[#02066F] mb-3">Employee Time Tracking</h2>
<p class="text-[16px]">
  One tap solution for simplifying and streamlining employee time logging and reporting.
</p>
</section>

<!-- Features Section -->
<section id="whatWeProvide" class="flex flex-col lg:flex-row p-4 pt-10 gap-12 items-center">
<img src="/main image.jpeg" alt="Main Feature" class="w-full lg:w-1/2 shadow-md" />

<div class="space-y-8 w-full lg:w-1/2">
  {#each features as { icon, title, desc }}
    <div class="flex items-start gap-6 sm:gap-8">
      <img src={`/${icon}`} alt={title} class="w-14 h-14 sm:w-18 sm:h-18 object-contain" />
      <div class="flex flex-col gap-2">
        <h4 class="text-xl sm:text-2xl font-semibold">{title}</h4>
        <p class="text-gray-800 text-base sm:text-lg">{desc}</p>
      </div>
    </div>
  {/each}
</div>
</section>

<!-- Contact Section -->
<section id="contact" class="pt-18">
<div class="max-w-2xl mx-auto px-4">
  <h2 class="text-[32px] sm:text-[40px] font-bold text-gray-900 text-center mb-2">Contact</h2>

  <form
    on:submit|preventDefault={validateForm}
    class="bg-white rounded-lg shadow-[0_0_20px_rgba(0,0,0,0.2)] p-6 sm:p-10 space-y-6"
  >
    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
      <input
        type="text"
        placeholder="First Name"
        bind:this={firstNameInput}
        class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold text-center focus:outline-none"
        required
      />
      <input
        type="text"
        placeholder="Last Name"
        bind:this={lastNameInput}
        class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold text-center focus:outline-none"
        required
      />
      <input
        type="email"
        placeholder="Email"
        bind:this={emailInput}
        class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold text-center focus:outline-none"
        required
      />
      <input
        type="text"
        bind:value={phone}
        placeholder="Phone"
        bind:this={phoneInput}
        on:input={formatPhoneNumber}
        class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold text-center focus:outline-none"
        required
      />
      <input
        type="text"
        placeholder="Street"
        bind:this={streetInput}
        class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold text-center focus:outline-none"
        required
      />
      <input
        type="text"
        placeholder="City"
        bind:this={cityInput} 
        class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold text-center focus:outline-none"
        required
      />
      <!-- <input
        type="text"
        placeholder="Zip"
        class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold text-center focus:outline-none"
        required
      /> -->

      <input
      type="text"
      placeholder="Zip"
      bind:value={zip}
      bind:this={zipInput}
      on:input={validateZip}
      class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold text-center focus:outline-none"
      required
    />

      <input
        type="text"
        placeholder="State"
        bind:this={stateInput}
        class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold text-center focus:outline-none"
        required
      />
    </div>

    <input
      type="text"
      placeholder="Subject"
      bind:this={subjectInput}
      class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold text-center focus:outline-none"
      required
    />

    <textarea
      placeholder="Message..."
      rows="4"
      bind:this={messageInput}
      class="border-2 border-[#02066F] rounded-[10px] w-full p-3 font-bold resize-none text-center focus:outline-none"
      required
    ></textarea>

    <button
      type="submit"
      class="w-full bg-[#02066F] text-lg cursor-pointer text-white font-semibold py-4 rounded-[10px] transition-colors"
    >
      Submit
    </button>
  </form>
  <!-- {#if showOverlay}
  <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center text-white text-xl">
    Processing...
  </div>
{/if} -->
{#if show}
  <div
    class="fixed inset-0 flex items-center justify-center z-50 " 
    style="background: rgba(0, 0, 0, 0.5)"
    on:click={handleBackgroundClick}
  >
    <div
      id="modal-content"
      class="bg-white rounded-lg shadow-lg max-w-lg w-full py-6  text-center"
    >
      <h2 class="text-xl text-gray-800 font-semibold mb-4">Thank You for Contacting Us!</h2> 
      <hr class="text-gray-400 w-full mx-auto mb-6">
      <p class="text-gray-800 font-semibold text-xl mb-6">
        We have received your message and will get back to you shortly.
      </p>
      <div class="flex justify-center">
        <img
          src="https://www.shutterstock.com/image-vector/blue-check-mark-icon-tick-260nw-787016416.jpg"
          alt="Checkmark"
          class="w-24 h-24 object-contain"
        />
      </div>
    </div>
  </div>
{/if}
</div>
</section>
