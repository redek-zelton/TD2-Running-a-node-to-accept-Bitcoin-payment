<template>
  <div class="about">
    <h1>BTCPayServer Test Button</h1>
  </div>
    <div>
    <form method="POST" action="https://testnet.demo.btcpayserver.org/apps/3V7dQqRCh8hBSGk2BDn5GZU6HQTb/pos">
    <input type="hidden" name="amount" value="1" />
    <button class="btnBTC" type="submit">Payment en Wallet BTC   1$ </button>
  </form>
  </div>

  <div>
    <form method="POST" action="https://testnet.demo.btcpayserver.org/apps/3V7dQqRCh8hBSGk2BDn5GZU6HQTb/pos">
    <input type="hidden" name="amount" value="10" />
    <button class="btnLND" type="submit">Payment via Lightning Network     10$ </button>
  </form>
  </div>
</template>
<script>
</script>
<style>
.btnBTC {
  margin-top: 10%;
  display: flex;
  align-content: center#b66912;
  justify-content: center;
  color: #fff;
  background-color: #198754;
  border-color: #198754;
  padding-bottom: 30px;
  height: 20%;
  width: 50%;
  font-size: 30px;
}
.btnLND {
  margin-top: 12%;
  display: flex;
  align-content: center;
  justify-content: center;
  color: #fff;
  background-color: #b66912;
  border-color: #b66912;
  padding-bottom: 30px;
  height: 20%;
  width: 50%;
  font-size: 30px;
}
</style>
