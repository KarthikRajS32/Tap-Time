<script>
  import EmployeeList from "../../components/Employee/EmployeeList.svelte";
  import Footer2 from "../../components/Footer/Footer2.svelte";
  import Navbar from "../../components/Header/Navbar.svelte";
  import { onMount } from 'svelte';
  
  let isAuthenticated = false;

  onMount(() => {
    // Client-side check
    const companyID = localStorage.getItem('companyID');
    if (!companyID) {
      window.location.href = '/login';
    } else {
      isAuthenticated = true;
    }
  });
</script>

{#if isAuthenticated}
  <Navbar/>
  <EmployeeList/>
  <Footer2/>
{/if}