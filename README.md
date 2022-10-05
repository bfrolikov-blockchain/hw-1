## Task 1
```diff
diff --git a/MultiSigWallet.sol b/MultiSigWallet.sol
index 6a777f2..cfcf7c5 100644
--- a/MultiSigWallet.sol
+++ b/MultiSigWallet.sol
@@ -91,6 +91,11 @@ contract MultiSigWallet {
         _;
     }
 
+    modifier validTransactionValue(uint value) {
+        require(value <= 66 ether);
+        _;
+    }
+
     /// @dev Fallback function allows to deposit ether.
     function()
         payable
@@ -289,6 +294,7 @@ contract MultiSigWallet {
     function addTransaction(address destination, uint value, bytes data)
         internal
         notNull(destination)
+        validTransactionValue(value)
         returns (uint transactionId)
     {
         transactionId = transactionCount;
```
## Task 2
```diff
diff --git a/ERC20.sol b/ERC20.sol
index 9d0c282..54b5490 100644
--- a/ERC20.sol
+++ b/ERC20.sol
@@ -208,6 +208,7 @@ contract ERC20 is Context, IERC20 {
     function _transfer(address sender, address recipient, uint256 amount) internal virtual {
         require(sender != address(0), "ERC20: transfer from the zero address");
         require(recipient != address(0), "ERC20: transfer to the zero address");
+        require(_getCurrentWeekDay() != 6, "Tokens cannot be transfered on Saturdays");
 
         _beforeTokenTransfer(sender, recipient, amount);
 
@@ -218,6 +219,10 @@ contract ERC20 is Context, IERC20 {
         emit Transfer(sender, recipient, amount);
     }
 
+    function _getCurrentWeekDay() private pure returns (uint8) {
+        return (block.timestamp / 1 days + 4) % 7;
+    }
+
     /** @dev Creates `amount` tokens and assigns them to `account`, increasing
      * the total supply.
      *
```
## Task 3
```diff
diff --git a/DividendToken.sol b/DividendToken.sol
index 651290f..77bd271 100644
--- a/DividendToken.sol
+++ b/DividendToken.sol
@@ -18,7 +18,7 @@ contract DividendToken is StandardToken, Ownable {
     event PayDividend(address indexed to, uint256 amount);
     event HangingDividend(address indexed to, uint256 amount) ;
     event PayHangingDividend(uint256 amount) ;
-    event Deposit(address indexed sender, uint256 value);
+    event Deposit(address indexed sender, uint256 value, bytes32 comment);
 
     /// @dev parameters of an extra token emission
     struct EmissionInfo {
@@ -37,9 +37,9 @@ contract DividendToken is StandardToken, Ownable {
         }));
     }
 
-    function() external payable {
+    function receiveEther(bytes32 comment) external payable {
         if (msg.value > 0) {
-            emit Deposit(msg.sender, msg.value);
+            emit Deposit(msg.sender, msg.value, comment);
             m_totalDividends = m_totalDividends.add(msg.value);
         }
     }
```