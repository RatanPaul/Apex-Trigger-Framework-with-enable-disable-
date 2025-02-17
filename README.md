# Apex Trigger Framework

A robust and scalable Apex Trigger Framework for Salesforce that allows enabling and disabling triggers dynamically using **Custom Metadata Types**.

## Features
- ✅ **Enable/Disable Triggers** dynamically without modifying code.
- ✅ **Centralized Trigger Management** via Custom Metadata.
- ✅ **Reusable & Scalable** structure for handling triggers efficiently.
- ✅ **Consistent Trigger Execution Flow** with separate handlers.

---

## 1. Setup Custom Metadata
Create a **Custom Metadata Type**: `Trigger_Control__mdt` with the following fields:

| Field Name       | Data Type | Description |
|------------------|----------|-------------|
| Trigger_Name__c  | Text     | Stores the trigger name |
| Is_Enabled__c    | Checkbox | Controls trigger execution (True = Enabled, False = Disabled) |

---

## 2. Framework Components

### **Trigger Handler Interface**
Defines a standard structure for trigger handlers.
```apex
public interface ITriggerHandler {
    void beforeInsert(List<SObject> newRecords);
    void beforeUpdate(Map<Id, SObject> oldRecords, Map<Id, SObject> newRecords);
    void beforeDelete(Map<Id, SObject> oldRecords);
    void afterInsert(List<SObject> newRecords);
    void afterUpdate(Map<Id, SObject> oldRecords, Map<Id, SObject> newRecords);
    void afterDelete(Map<Id, SObject> oldRecords);
    void afterUndelete(List<SObject> restoredRecords);
}
```

### **Base Trigger Handler**
Handles trigger enable/disable logic.
```apex
public abstract class BaseTriggerHandler implements ITriggerHandler {
    private Boolean isTriggerEnabled;

    public BaseTriggerHandler(String triggerName) {
        this.isTriggerEnabled = checkTriggerEnabled(triggerName);
    }

    private Boolean checkTriggerEnabled(String triggerName) {
        Trigger_Control__mdt setting = [SELECT Is_Enabled__c FROM Trigger_Control__mdt WHERE Trigger_Name__c = :triggerName LIMIT 1];
        return setting != null ? setting.Is_Enabled__c : true;
    }

    public Boolean isEnabled() {
        return isTriggerEnabled;
    }
}
```

### **Trigger Utility**
Utility class for checking trigger status.
```apex
public class TriggerUtility {
    public static Boolean isTriggerEnabled(String triggerName) {
        try {
            Trigger_Control__mdt setting = [SELECT Is_Enabled__c FROM Trigger_Control__mdt WHERE Trigger_Name__c = :triggerName LIMIT 1];
            return setting != null ? setting.Is_Enabled__c : true;
        } catch (Exception e) {
            return true;
        }
    }
}
```

### **Example Trigger Handler (For Account)**
Handles Account triggers.
```apex
public class AccountTriggerHandler extends BaseTriggerHandler {
    public AccountTriggerHandler() {
        super('AccountTrigger');
    }

    public override void beforeInsert(List<SObject> newRecords) {
        if (!isEnabled()) return;
        System.debug('Before Insert Logic');
    }
}
```

### **Example Trigger (Account Trigger)**
```apex
trigger AccountTrigger on Account (
    before insert, before update, before delete,
    after insert, after update, after delete, after undelete
) {
    AccountTriggerHandler handler = new AccountTriggerHandler();

    if (Trigger.isBefore) {
        if (Trigger.isInsert) handler.beforeInsert(Trigger.new);
    }
}
```

---

## 3. How to Enable/Disable a Trigger
1. Navigate to **Custom Metadata Types** → `Trigger_Control__mdt`.
2. Click **Manage Records**.
3. Create or Edit a record:
   - **Trigger Name:** `AccountTrigger`
   - **Is Enabled:** `True` (Enable) or `False` (Disable)

---

## 4. Benefits of This Framework
✅ **Dynamic Control:** Change trigger behavior without deployments.
✅ **Reusability:** Standard structure for all triggers.
✅ **Scalability:** Easily extend to other objects.

---

## 5. Contributing
Pull requests are welcome! For major changes, please open an issue first to discuss what you would like to change.

---

## 6. License
This project is open-source and available under the MIT License.

