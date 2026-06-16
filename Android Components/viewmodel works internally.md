ViewModel survives configuration changes because it is stored inside a ViewModelStore. 
During Activity recreation, Android preserves the ViewModelStore inside a NonConfigurationInstances object and restores it to the new Activity instance.
When ViewModelProvider requests the ViewModel again, it finds the existing ViewModel in the restored ViewModelStore and returns the same instance instead of creating a new one.
