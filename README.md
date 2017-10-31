TASK DATE: 31.10.2017 - FINISHED: 

TASK SHORT DESCRIPTION: 1177 (logging an activity when sending 1-2-1 emails via 'send email' button on profile)

GITHUB REPOSITORY CODE: feature/task-1177-logging-sent-email-through-profile

ORIGINAL WORK: https://github.com/BusinessBecause/network-site/tree/feature/task-1177-logging-sent-email-through-profile


CHANGES
 
	IN FILES: 
		
		\network-site\addons\default\modules\network_settings\language\english\network_settings_lang.php
		
			ADDED CODE: 
			
				$lang['activity_tracker_email_was_sent_name'] = 'Was sent: %s<br>Subject: %s<br>From: %s<br>To: %s';
	
	
		\network-site\addons\default\modules\network_settings\controllers\members.php	
		
			CHANGED/ADDED CODE: 
			
				/************ Insert details to activity tracker *************/
				$this->load->model('network_settings/activity_tracker_m');
				$this->load->model('bbusers/profile_m');
		
				$user_info = $this->profile_m->get_names($id);
				$email_from = $user_info[0]['salutation'] . " " . $user_info[0]['first_name'] . " " . $user_info[0]['last_name'];
				$time = time();
				$insert_data = array(
					'updated' => $time, 
					'date' => $time, 
					'type' => '0', 
					'name' => sprintf(lang('activity_tracker_email_was_sent_name'), date('d-m-Y H:i:s', $time), $subject, $email_from, $email_to), 
					'result' => $email_message, 
					'assigned_to' => $id, 
					'completed' => 1			
				);			
				
				$new_tracker_id = $this->activity_tracker_m->create_activity($insert_data);
				
				$this->activity_tracker_m->set_logged_by($new_tracker_id, $id, $time);
				$this->activity_tracker_m->assign_user_to_activity($new_tracker_id, $id, $time);
						
			
		\network-site\addons\default\modules\network_settings\models\activity_tracker_m.php	
			
			CHANGED/ADDED CODE 1:  
				
				FROM: 
				
					public function assign_user_to_activity($activity_id, $user_id) {
						if (!$this->db->where('activity_id', $activity_id)->where('user_id', $user_id)->get('default_activity_tracker_user_association')->result())
						{
							$this->db->query("
							  INSERT INTO `default_activity_tracker_user_association` (`activity_id`, `user_id`, `created_on`)
							  VALUES (".$activity_id.", ".$user_id.", ".time().")");
						}
					}
					
					
				TO: 
				
					 public function assign_user_to_activity($activity_id, $user_id, $time = 0) {
						if ($time == 0) $time = time();
						if (!$this->db->where('activity_id', $activity_id)->where('user_id', $user_id)->get('default_activity_tracker_user_association')->result())
						{
							$this->db->query("
							  INSERT INTO `default_activity_tracker_user_association` (`activity_id`, `user_id`, `created_on`)
							  VALUES (".$activity_id.", ".$user_id.", ".$time.")");
						}
					}
					
					
			CHANGED/ADDED CODE 2:  
				
				FROM: 
				
					public function create_activity_if_does_not_exist($activity_title) {
						$result = $this->db->query("
							SELECT `id`
							FROM `default_activity_tracker`
							WHERE `name`='".$activity_title."'")->result();
						if(count($result) == 0) {
							$this->db->query("
							  INSERT INTO `default_activity_tracker` (`updated`, `date`, `type`, `name`, `assigned_to`, `completed`)
							  VALUES (".time().", ".time().", 5, '".$activity_title."', 1, 1)");
							return $this->db->insert_id();
						} else {
							return $result[0]->id;
						}
					}
					
					
				TO: 
				
					public function create_activity_if_does_not_exist($activity_title, $type = '5', $assigned_to = 1, $completed = 1) {
						$hits = $this->db->query("
												SELECT `id`
												FROM `default_activity_tracker`
												WHERE `name`='".$activity_title."'"
											)->result();
						if(count($hits) == 0) 
						{
							$time = time();
							$insert_data = array(
								'updated' => $time, 
								'date' => $time, 
								'type' => '0', 
								'name' => $activity_title, 
								'assigned_to' => $assigned_to, 
								'completed' => $completed			
							);
							return $this->create_activity($insert_data);
						} 
						else 
						{
							return $hits[0]->id;
						}
					}

					/*
						* Create a new record in default_activity_tracker table
						*
						* @input
						*	- $new_record : array
						*	- fields can be: id, updated, date, type, name, result, tracker_type, import_ref, assigned_to, completed
						*
						* @return
						*	- $id: id of last insert activity
					*/
					public function create_activity($new_record = array()) {
						
						if (count($new_record) > 0) 
						{
							$this->db->insert('default_activity_tracker', $new_record);
							return $this->db->insert_id();
						}
						else 
						{
							return 0;
						}
						
					}
