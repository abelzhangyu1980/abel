# IBM FileNet Knowledge Collections
## Local Group
[Link] (https://www.ibm.com/docs/en/filenet-p8-platform/5.5.x?topic=authorization-v557-later-understanding-local-groups)
local group is a new feature introduced since v5.5.7. the idea behind is external user repository is lack of dynamicity. hence FileNet provides another alternative to allow application to create FileNet-Managed groups (basically it's a federated repository from websphere perspective).
and allow application to use either ICN? or API to CRUD groups. 
API Samples --TBD

## Role based access control
[Link] (https://www.ibm.com/docs/en/filenet-p8-platform/5.5.x?topic=authorization-understanding-role-based-access)
instead of using static groups to control who can access which object. FileNet provides Role based (both static and dynamic) access control. 
### static role based access control
static roles are roles which members are pre-populated. static role can contain other roles, groups or users.
### dynamic role based access control
unlick static role, dynamic role does not define members. instead, members are determined during runtime via calling RoleMembershipHandler implementation. [Link] (https://www.ibm.com/docs/en/filenet-p8-platform/5.5.x?topic=access-creating-dynamic-role)
