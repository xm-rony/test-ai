Background
Right now the service DB is mainteining a table to correlate a session record (game_sessions) with a record of the session_configurations table that containse the app config properties that the session got initialized. 

So far the session_configurations table has only the foreign key to the game_sessions and the volatility_factor ,that is being updated based on the helm parameter after service's re-depoyment.

Later on extra configuration parameters are going to be added, and administration endpoints must handle the update process, whenever it is needed without redeployment.

A UI will manage the config params update.

Solution
To proper handle this functionality, the config table must be updated to keep the current version of params that the config table was mainteining at the time that the session initialized. 

To achieve this:

the session_configurations must be replaced by the config param table.
This table will maintein only a unique id and the config params (volatility for the time being).
a default record with volatility_factor equal to 1 must be the initial record
The game_sessions table must be replace the session_configuration_id with the new foreign key of config_param table, and all the current records must be updated to have the default record.
The service must cache the config parameters so as to be easiliy available, and whenever a new session is created the record will have to store the config params that are currently cached along to the rest session parameters
The config param values (volatility) must be used whenever it is needed (
an admin endpoint must be added to a admin controler that will allow to create a new record on the config_params , introducing a new value of the volatility factor and triggering a process to update the cache


Accepance Criteria:

an endpoint must be exposed to the service admins, allowing them to update (actually create a new record on the config param table) the volatility factor
the update config params must be used for any new session created after an config param update
all game sessions must maintenain the config param information based on which all bets were being created and evalutated.
