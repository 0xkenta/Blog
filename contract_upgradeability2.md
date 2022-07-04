<!-- # ロジックコントラクトを変更する際の注意点
you cannot change the order in which the contract state variables are declared, nor their type. 

If you need to introduce a new variable, make sure you always do so at the end

And if you remove a variable from the end of the contract, note that the storage will not be cleared. A subsequent update that adds a new variable will cause that variable to read the leftover value from the deleted one. -->