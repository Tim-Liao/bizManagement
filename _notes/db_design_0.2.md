------------------------------------ system task config ----------------------------------
chooce 1 : relational data stucture below to keep the task work flow graph
chooce 2 : use seriliable file to keep the graph the task work flow graph
// 1 : many db access, but more availble to change, quick access for every step
// 2. better to store the SESSION(achieved by Redis in mricroservice), but also need to retrive graph to get current step
//suggestion : chooce 1

-------------------------------------------------------------------------------------------
# wf_process_definition
	* id : int(9) | primary key
	* name : varchar(90)
	* trigger : varchar(45) | unique 
	* root_activity : int(9) | referenced from task_process_activity.id
	* is_active : varchar(1)


# wf_activity
	* id : int(9) | priamry key
	* name : varchar(90) 
	* participant_position : int(9) | referenced from hr_position_id.id (M2O)
	* request_task : varchar(45) | enum
	* activity_type : varchar(20) | enum | manual,automated
	* condition : varchar(90) | criteria expression


# wf_process_routing
	* id :  int(9) | priamry key
	* from_activity : int(9) | referenced from wf_activity.id
	* to_activity : int(9) | referenced from wf_activity.id
	* routing_type : varchar(20) | enum | sequential_route, AND_split, OR_spilit, AND_join, OR_join


# wf_process_instance
	* id : int(9) | primary key
	* description : varchar(150)
	* process_definition : int(9) | referenced from wf_process_definition.id 
	* create_time : datetime
	* status : varchar(20) | enum | initiated, running, suspended, complete, terminated
	// to record the current 


# wf_activity_instance
	* id : int(9) | primary key
	* description : varchar(150)
	* activity : int(9) | referenced from wf_activity.id 
	* process_instance : int(9) | referenced from wf_process_instance.id
	* create_time : datetime
	* status : varchar(20) | enum | pending, complete, closed, running
	* update_time : datetime


# wf_worklists
	* id : int(12) | primary key
	* activity_instance : int(9) | referenced from wf_activity_instance.id
	* user_id :
	//reassign when employee change . change db table

----------------------------------- user microservice --------------------------------
1. manager the access control
	// use "Resource-based access control" idea to achieve the access control
	// user attaches to roles, permissions attached to the role -> get user's permission by retrieve the permissions
	// to speed up, after account login -> save all permissions in SESSION(Redis)
	// the idea is referenced from Shiro (a famous access control framework)
--------------------------------------------------------------------------------------

# user_info
	* id : int(9) | primary key
	* name : varchar(45) 
	* email : varchar(45) | unique | used as login account
	* phone : varchar(45)
	* avatar : blob | binary
	* password : varchar(255) | save code encrypted by MD5 
	* pwd_salt : varchar(30) |  random string, as salt for MD5
	* created_time : datatime
	* last_login_time : datetime
	* is_active : varchat(1)
	* job_status : varchar(20) | enum | in_holiday, part_time, full_time, retired, dismissed
	//


# user_role
	* id : int(9) | primary key
	* name : varchar(90) | unique
	* description : text


# user_permission
	* id : int(9) | primary key 
	* name : varchar(90) 
	* resource_type : varchar(30) | enum 
	* resource_url : varchar(60)
	* permission_expression : varchar(150)


# user_relation_info_role
	* user_id : int(9) | primary key
	* user_role_id : int(9) | primary key


# user_relation_role_permission
	* user_role_id : int(9) | primary key
	* user_permission_id : int(9) | primary key


# user_relation_info_permission
	* user_id : int(9) | primary key
	* user_permission_id : int(9) | primary key

// add user directly to permission

----------------------------------- HR microservice --------------------------------

# hr_department
	* id : int(9) | primary key
	* name : varchar(90) | unique
	* address : varchar(255)
	* phone : varchar(30)


# hr_enum_position
	* id : int(9) | primary key
	* name : varchar(90) | unique
	* specific_department : int(9) | referenced from hr_department.id 


# hr_user_position (hr_relation_user_department_position)
    * id : int(9) | primary key
	* user_id : int(9) | index | referenced from user_info.id
	* department_id : int(9) | index | referenced from hr_department.id
	* position_id : int(9) | index | referenced from hr_department.id


# hr_employee_document
	* id : int(11) | primary key
	* name : varchar(60)
	* user_id : int(9) | referenced from user_info.id (M2O)
	* document_type : varchar(30) | enum
	* hard_copy_only : varchar(1)
	* created_time : datetime


# hr_document_consult_history
	* id : int(13) | primary key
	* consult_type : varchar(30) | enum (creation/modification/read_only)
	* requestor_id : int(9) | referenced from user_info.id
	* approver_id : int(9) | referenced from user_info.id
	* comment : text
	* consult_time : datatime


# hr_employee_timesheet
	* id : int(15) | primary key
	* user_id : int(9) | referenced from user_info.id
	* work_date : date 
	* start_from : time
	* end_at : time


# hr_employee_salary
	* id : int(12) | primary key
	* user_id : int(9) | referenced from user_info.id
	* salary_type : varchar(20) | enum | daily, weekly, monthly, yearly
	* work_from : datetime
	* work_to : datetime
	* basic_salary : double(5,2)
	* bonus : double(5,2)
	* pension : double(5,2)
	* insurance 
	...
	...
	...

------------------------------------ inventory microservice ----------------------------------------

# inventory_commidity
	* id : int(9) | primary key
	* name : varchar(60)
	* commidity_type : varchar(30) | enum | raw/ready for production/semi-finished / finished
	* quantity_unit : varchar(30) | enum | not null
	* processing_period : int(6)  | unit:day | not null | in processing com, need processing table to calculate avg


# inventory_supplier (vendor)
	* id : int(9) | primary key
	* supplier_name : varchar(60) | unique | not null
	* address : varchar(150)
	* description : text
	* business license : blob


# inventory_supplier_contact
	* id : int(9) | primary key
	* belong_supplier : int(9) | referenced from inventory_supplier.id (M2O)
	* name : varchar(30) 
	* phone_number : varchar(30) 
	* email_address : varchar(30)


# inventory_relation_commidity_supplier
	* commidity_id : int(9) | primary key
	* supplier_id : int(9) | primary key
	* purchase_period : int(6) | unit:day | not null



# inventory_stock_in 
	* id : int(11) | primary key
	* batch_number : varchar(30) | unique | index
	* commidity_id : int(9) | referenced from inventory_commidity.id  //type of commidity
	* supplier_id : int(9) | referenced from inventory_supplier.id
	* entry_time : datatime
	* receive_user :  int(9) | referenced from user_info.id
	//* account_transation : int(17) | referenced from account_transaction.id


# inventory_commidity_units
	* serial_number : int(15)   //for search key
	* sku : int(15) |
	* commidity_id : int(9) | referenced from inventory_commidity.id
	* stock_in_id : int(11) | referenced from inventory_entry.id
	* commidity_status : varchar(30) | enum | in_stock/ saled / ready_to_product / broken /  picked_for_process /
	* cost_unit : 

// lot number

# inventory_picking
	* id : int(15) | primary key
	* commidity_units_id : int(9) | referenced from inventory_commidity_units.id
	* picked_time : datatime
	* picked_user : int(9) | referenced from user_info.id
	* approved_user : int(9) | referenced from user_info.id
	* picking_purpose : varchar(30) | enum | sell/processing/cleaning
	//* account_transation : int(17) | referenced from account_transaction.id


------------------------------------ accountancy microservice ----------------------------------------

# account_enum_title
	* id : int(9) | primary key
	* name : varchar(30) | unique
	* title_level : int(1)


# account_transaction
	* id : int(17) | primary key
	* debit_title : int(9) | referenced from account_enum_title.id
	* credit_tile : int(9) | referenced from account_enum_title.id
	* explanation : varchar(150) 
	* create_time : datatime
	* create_user : int(9) | referenced from user_info.id
	* transation_status : varchar(30) | enum | pending/comformed
	* checked_user : int(9) | referenced from user_info.id


# account_employee_salary
// type: monthly, hourly,.....
// calculate tax, amount, 



# account_employee_benifit
// user_id
// contract_type : ... T4......
// for every year
// to keep a table to calcaulate the salary 
// reference from T4A



	---------------------------- invoice microservice -------------------------------------------------