# bluetoothle-codenameone
Bluethooth LE Library for [Codename One](https://github.com/codenameone/CodenameOne) apps.
This Library is based on the excellent cordova plugin from https://github.com/randdusing/cordova-plugin-bluetoothle

##Integration
1)Build the project <br/>
2)Place the CN1Bluethooth.cn1lib file in your CN1 project lib. <br/>
3)Add the CN1JSON.cn1lib file in your CN1 project lib. (https://github.com/shannah/CN1JSON/) <br/> 
4)Right click on your CN1 project and select "Refresh Libs" then clean build your project.

## Sample of Usage

```java
        final Bluetooth bt = new Bluetooth();
        Form main = new Form("Bluetooth Demo");
        main.setLayout(new BoxLayout(BoxLayout.Y_AXIS));
        main.add(new Button(new Command("enable bluetooth") {

            @Override
            public void actionPerformed(ActionEvent evt) {

                try {
                    if (!bt.isEnabled()) {
                        bt.enable();
                    }
                    if (!bt.hasPermission()) {
                        bt.requestPermission();
                    }
                } catch (IOException ex) {
                    ex.printStackTrace();
                }
            }
        }));
        main.add(new Button(new Command("initialize") {

            @Override
            public void actionPerformed(ActionEvent evt) {
                try {
                    bt.initialize(true, false, "bluetoothleplugin");
                } catch (IOException ex) {
                    ex.printStackTrace();
                }
            }
        }));
```


		public void scan() 
		{
		
			if(LastConnectedUUID != null)		// String LastConnectedUUID, used to track last connected
			{
				try 
				{
					if(!Display.getInstance().isSimulator()) 
					{
						bt.disconnect(Parent.LastConnectedUUID);
					}
				} catch (IOException ex) {
					LogIt("scan() " + ex.getMessage());
				} catch (Exception ex) {
					LogIt("scan() " + ex.getMessage());
					
				}
			}

			
			FoundDevices.clear();	//     public List<String> FoundDevices

			try 
			{
				if(!Display.getInstance().isSimulator()) 
				{
					bt.startScan(new ActionListener() 
						{  
						@Override
							public void actionPerformed(ActionEvent evt) 
							{
								ScanDeviceFound(evt);
							}
						}, 
						null, 
						true, 
						Bluetooth.SCAN_MODE_LOW_POWER, 
						Bluetooth.MATCH_MODE_STICKY,
						Bluetooth.MATCH_NUM_MAX_ADVERTISEMENT, 
						Bluetooth.CALLBACK_TYPE_ALL_MATCHES);
				}
			} catch (IOException ex) {
					LogIt(ex.getMessage());
			} 

			Timer ScanTimer = new Timer();
			ScanTimer.schedule(new TimerTask() {
						@Override
						public void run() 
						{
							try 
							{
								if(!Display.getInstance().isSimulator())
									bt.stopScan();
							} catch (IOException ex) {
								LogIt(ex.getMessage());
							}                
						}
					}, ScanTimeout);
		}


		
		void ScanDeviceFound(ActionEvent evt)
		{
			try {
				JSONObject res = (JSONObject) evt.getSource();
				if (res.getString("status").equals("scanResult") && res.getString("name").equals("SigmaTau1")) {

						FoundDevices.add(res.getString("address"));					// add uuid to list of found devices
						String[] data = new String[Parent.FoundDevices.size()];		// covert array to list of strings
						FoundDevices.toArray(data);									
						SettingsForm_ScanListPicker.setStrings(data);				// add items to picker
						
					}
				}
			} catch (JSONException ex) {
				LogIt(ex.getMessage());
			}
		}

		

		void Connect(String uuid)
		{
			try {

				if(!Display.getInstance().isSimulator()) 
				{
					bt.connect(
						new ActionListener() 
						{
							@Override
							public void actionPerformed(ActionEvent evt) {
								ConnectCallback((JSONObject) evt.getSource());
							}    
						},uuid);
				}
			} catch (IOException ex) {
				LogIt("Connect: exception");
				LogIt(ex.getMessage());
			}
		}

		void ConnectCallback(JSONObject res)
		{
			try {
				if (res.getString("status").equals("connected")) 
				{
					Connected = true;
					
					Preferences.set("LastConnectedUUID",Parent.LastConnectedUUID);
					try {
						bt.discover(
							new ActionListener() 
							{
								@Override
								public void actionPerformed(ActionEvent evt) {
									LogIt(evt.getSource().toString());			// this has the services the device provides and other information in json

								}    
							},LastConnectedUUID);
					} catch (IOException ex) {
						LogIt("ConnectCallback:bt.discover: exception");
						LogIt( ex.getMessage());
					}
					
				}
				if (res.getString("status").equals("disconnected")) 
				{
					Connected = false;
				}
			} catch (JSONException ex) {
				LogIt("ConnectCallback exception: ");
				LogIt( ex.getMessage());
			}
		}

	
	    void SubscriptToNotifications()
		{
			try {
                bt.subscribe(
                    new ActionListener() 
                    {
                        @Override
                        public void actionPerformed(ActionEvent evt) {
                            NotificationReceived((JSONObject) evt.getSource());
                        }    
                    }, LastConnectedUUID, "1818", "2A63");	// Service and Characteristic
            } catch (IOException ex) {
                LogIt("SubscriptToNotifications: subscribe (exception)" + ex.getMessage() );
            }

		}
	

	    void NotificationReceived(JSONObject res)
		{
			try 
			{
				String b64 = res.getString("value");
				if (b64.length() > 0)
				{
					try {
						byte[] decodedString = Base64.decode(b64.getBytes("UTF-8"));
						/* extract bytes here */
						
					} catch (UnsupportedEncodingException ex) {

					}
						
				}
			}
			catch (JSONException ex) {
			}
		}

		void CalibrateSetValue(int Message, int value)
		{
			byte[] array = new byte[4];		// send four btes

			
			array[0] = 0;
			array[1] = (byte)Message;
			array[2] = (byte)(value / 256);
			array[3] = (byte)(value & 0xff);
			String b64 = Base64.encode(array);
				
				try {
					bt.write(new ActionListener() 
						{
							@Override
							public void actionPerformed(ActionEvent evt) {

							}    
						}, Parent.LastConnectedUUID, "1818", "FF01", b64, false);
				}
				catch (IOException ex) {

				}
		
		}

	
	
	
##Credits
1. Steve Hannah - for the https://github.com/shannah/CN1JSON
2. Rand Dusing - for the https://github.com/randdusing/cordova-plugin-bluetoothle
