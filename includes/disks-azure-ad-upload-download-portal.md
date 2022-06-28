---
 title: include file
 description: include file
 services: storage
 author: roygara
 ms.service: storage
 ms.topic: include
 ms.date: 06/30/2022
 ms.author: rogarana
 ms.custom: include file
---
If you're using [Azure Active Directory (Azure AD)](../articles/active-directory/fundamentals/active-directory-whatis.md) to control resource access, you can now use it to restrict uploads and downloads of Azure managed disks. This feature is currently in preview. When a user attempts to upload or download a disk, Azure validates the identity of the requesting user in Azure AD, and confirms that user has the required permissions. At a higher level, a system administrator could set a policy at the Azure account or subscription level, to ensure that all disks and snapshots must use Azure AD for uploads or downloads.

### Restrictions
[!INCLUDE [disks-azure-ad-upload-download-restrictions](disks-azure-ad-upload-download-restrictions.md)]
- To download a VHD that is using Azure AD to restrict access, you must access the Azure portal from this link: [https://aka.ms/dataAccessAuthenticationMode](https://aka.ms/dataAccessAuthenticationMode)

### Prerequisites
[!INCLUDE [disks-azure-ad-upload-download-prereqs](disks-azure-ad-upload-download-prereqs.md)]

### Assign RBAC role

To access managed disks secured with Azure AD, the requesting user must have either the [Data Operator for Managed Disks](../articles/role-based-access-control/built-in-roles.md#data-operator-for-managed-disks) role, or a [custom role](../articles/role-based-access-control/custom-roles-portal.md) with the following permissions: 

- **Microsoft.Compute/disks/download/action**
- **Microsoft.Compute/disks/upload/action**
- **Microsoft.Compute/snapshots/download/action**
- **Microsoft.Compute/snapshots/upload/action**

For detailed steps on assigning a role, see [Assign Azure roles using the Azure portal](../articles/role-based-access-control/role-assignments-portal.md). To create or update a custom role, see [Create or update Azure custom roles using the Azure portal](../articles/role-based-access-control/custom-roles-portal.md).

### Enable data access authentication mode

You'll need to enable **data access authentication mode** to restrict access to the disk. You can either enable it when creating the disk, or on the **Disk Export** page on the disk after it's been created. In order to enable **data access authentication mode** you must access the Azure portal from the following link: [https://aka.ms/dataAccessAuthenticationMode](https://aka.ms/dataAccessAuthenticationMode)

:::image type="content" source="media/disks-azure-ad-upload-download-portal/disks-data-access-auth-mode.png" alt-text="Screenshot of a disk's data access authentication mode checkbox, tick the checkbox to restrict access to the disk." lightbox="media/disks-azure-ad-upload-download-portal/disks-data-access-auth-mode.png":::