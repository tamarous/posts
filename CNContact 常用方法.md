##CNContact 常用方法
引入Contact框架

	import ContactsUI
	
获得当前应用的授权状态
	
	class func authorizationStatusForEntityType(_ entityType: CNEntityType) -> CNAuthorizationStatus
	
如果没有进行授权，则使用下列语句来获得授权
	
	func requestAccessForEntityType(_ entityType: CNEntityType, completionHandler completionHandler: (Bool, NSError?) -> Void)


用Switch语句进行判断：

	switch CNContactStore.authorizationStatusForEntityType(.Contacts) {
    	case .Authorized:
        	createContact()
    	case .NotDetermined:
        	store.requestAccessForEntityType(.Contacts) {
            	succeeded, err in 
            	guard err == nil && succeeded else {
                	return
            	}
            	self.createContact()
        	}
    	default:
        print("Not handled")
	}
	
创建一个Contact Store实例：
	
	var store = CNContactStore()
	
	
###创建联系人：
	
	let contact = CNMutableContact()
	contact.imageData = NSData()

	contact.givenName = "Zewei"
	contact.familyName = "Wang"

	let homeEmail = CNLabeledValue(label: CNLabelHome, value: 	"532487608@qq.com")
	let workEmail = CNLabeledValue(label: CNLabelWork, value: 	"hiwangzewei@gmail.com")


	contact.emailAddresses = [homeEmail,workEmail];

	contact.phoneNumbers = 		[CNLabeledValue(label:CNLabelPhoneNumberiPhone,value: CNPhoneNumber(stringValue: "15664601205"))]

	let homeAddress = CNMutablePostalAddress()

	homeAddress.street = "Tai Bai Nan Lu"
	homeAddress.city = "Xian"
	homeAddress.state = "Shannxi"
	homeAddress.country = "China"

	homeAddress.postalCode = "710126"
	contact.postalAddresses = [CNLabeledValue(label: CNLabelHome, value: homeAddress)]

	let birthday = NSDateComponents()

	birthday.day = 1
	birthday.month = 4
	birthday.year = 1996

	contact.birthday = birthday
	
	contact.jobTitle = "Bar Manager"
	contact.organizationName = "Best English Bar"
	contact.departmentName = "Food and Beverages"
	
	let factbookProfile = CNLabeledValue(label:"FaceBook",value:CNSocialProfile(urlString:nil, username:"tamarous", userIdentifier:nil, service:CNSocialProfileServiceFacebook))
	contact.socialProfiles = [facebookProfile];

	let saveRequest = CNSaveRequest()

	saveRequest.addContact(contact, toContainerWithIdentifier: nil)
	do {
		try store.execureSaveRequest(request)
		print("Successfully added the contact")
	} catch let err {
		print("Failed to save to the contact.\(err)")
	}
	
###搜寻联系人：
	
	//way 1
	func enumerateContactsWithFetchRequest(_ fetchRequest: CNContactFetchRequest, usingBlock block: (CNContact, UnsafeMutablePointer<ObjCBool>) -> Void) throws
	
	//way 2
	func unifiedContactWithIdentifier(_ identifier: String, keysToFetch keys: [CNKeyDescriptor]) throws -> CNContact
	
	//way 3
	func unifiedContactsMatchingPredicate(_ predicate: NSPredicate, keysToFetch keys: [CNKeyDescriptor]) throws -> [CNContact]
	
寻找特定的联系人。假设现在要寻找一个名为Tamarous的联系人，那么我们首先要创建一个Predicate：
	
	let predicate = CNContact.predicateForContactsMatchingName("Tamarous")
	
然后寻找的范围是所有的family name和given name中含有Tamarous的人。

	let toFetch = [CNContactGivenNameKey, CNContactFamilyNameKey]
	
用上面的方法2来寻找：
	
	do {
    	let contacts = try store.unifiedContactsMatchingPredicate(predicate, keysToFetch: toFetch)

    	for contact in contacts {
       	print(contact.givenName)
        	print(contact.familyName)
        	print(contact.identifier)
    	}
	} catch let err {
    	print(err)
	}
	
方法1和方法2是类似的，只不过是由fetchKeys初始化了FetchRequest：

	let toFetch = [CNContactGivenNameKey]
	let request = CNContactFetchRequest(keysToFetch:toFetch)
	
	do{
    	try store.enumerateContactsWithFetchRequest(request) {
        	contact, stop in 
        	print (contact.givenName)
        	print (contact.familyName)
        	print (contact.identifier)
    	}
	} catch let err {
    	print(err)
	}
	
但是这两种方法得到的contacts是不同的。方法2中的contacts是满足了predicate的contact的集合，而方法1中的contacts则是所有的contact

一般来说，这种操作应该放在一个单独的进程中来进行，因此可以这样写：

	NSOperationQueue.addOperationWithBlock{
		[unowned store] in 
		do {
			...
			...
		} catch let err {
			...
		}
	}

###修改联系人信息：

假设我们要修改一个联系人"John"的电子邮件。首先，我们先判断他是否已经存储了这个新的邮件地址：
	
	let newEmailAdd = "newEmail@add.com"
	
	for email in contact.emailAddresses {
		if email.value as! String == newEmailAdd {
			print("This contact already has this email")
			return
		}
	}
	
	
如果没有的话，先创建一个可变的Contact实例：

	let john = contact.mutableCopy() as! CNMutableContact
	let emailAddress = CNLabeledValue(label:CNLabelWork, value:newEmailAdd)
	john.emailAddresses.append(emailAddress)
	
将更改保留下来：

	request.updateContact(john)
	try store.executeSaveRequest(request)
	print("Successfully added the email")
	
	
###删除联系人：

	let mutableContact = contact.mutableCopy() as! CNMutableContact
	request.deleteContact(mutableContact)
	
	do {
		try store.executeSaveRequest(request)
		print("Success, you deleted the contact")
	} catch let err {
		print("Error = \(err)")
	}
	
	
	
	
	
	
