# Go SDK

The ProfitBricks Client Library for [Go](https://www.golang.org/) provides you with access to the ProfitBricks REST API. It is designed for developers who are building applications in Go.

This guide will walk you through getting setup with the library and performing various actions against the API.

# Table of Contents
* [Concepts](#concepts)
* [Getting Started](#getting-started)
* [Installation](#installation)
* [How to: Create Data Center](#how-to-create-data-center)
* [How to: Delete Data Center](#how-to-delete-data-center)
* [How to: Create Server](#how-to-create-server)
* [How to: List Available Images](#how-to-list-available-images)
* [How to: Create Storage Volume](#how-to-create-storage-volume)
* [How to: Update Cores and Memory](#how-to-update-cores-and-memory)
* [How to: Attach or Detach Storage Volume](#how-to-attach-or-detach-storage-volume)
* [How to: List Servers, Volumes, and Data Centers](#how-to-list-servers-volumes-and-data-centers)
* [Reference](#reference)
  * [Data Centers](#data-centers)
    * [Types](#type-datacenter)
    * [List Data Centers](#list-data-centers)
    * [Retrieve a Data Center](#retrieve-a-data-center)
    * [Create a Data Center](#create-a-data-center)
    * [Update a Data Center](#update-a-data-center)
    * [Delete a Data Center](#delete-a-data-center)
  * [Servers](#servers)
    * [Types](#type-server)
    * [List Servers]()
    * [Retrieve a Server]()
    * [Create a Server]()
    * [Update a Server]()
    * [Delete a Server]()
    * [Reboot a Server]()
    * [Start a Server]()
    * [Stop a Server]()
    * [Attach a CDROM]()
    * [Detach a CDROM]()
    * [List attached CDROMs]()
    * [Get attached CDROM]()
  * [Volumes](#volumes)
    * [Types](#type-volume)
    * [List Volumes]()
    * [Retrieve a Volume]()
    * [Update a Volume]()
    * [Create a Volume]()
    * [Delete a Volume]()
  * [NICs](#servers)
    * [Types](#type-nic)
    * [Create a NIC]()
    * [Update a NIC]()
    * [Delete a NIC]()
* [Example](#example)
* [Support](#support)


# Concepts

The Go SDK wraps the latest version of the ProfitBricks REST API. All API operations are performed over SSL and authenticated using your ProfitBricks portal credentials. The API can be accessed within an instance running in ProfitBricks or directly over the Internet from any application that can send an HTTPS request and receive an HTTPS response.

# Getting Started

Before you begin you will need to have [signed-up](https://www.profitbricks.com/signup) for a ProfitBricks account. The credentials you setup during sign-up will be used to authenticate against the API.

Install the Go language from: [Go Installation](https://golang.org/doc/install)

The `GOPATH` environment variable specifies the location of your Go workspace. It is likely the only environment variable you'll need to set when developing Go code. This is an example of pointing to a workspace configured underneath your home directory:

```
mkdir -p ~/go/bin
export GOPATH=~/go
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOBIN
```

# Installation

The following go command will download `profitbricks-sdk-go` to your configured `GOPATH`:

```go
go get "github.com/profitbricks/profitbricks-sdk-go"
```

The source code of the package will be located at:

	$GOBIN\src\profitbricks-sdk-go

Create main package file *example.go*:

```go
package main

import (
	"fmt"
)

func main() {
}
```

Import GO SDK:

```go
import(
	"github.com/profitbricks/profitbricks-sdk-go"
)
```

Add your credentials for connecting to ProfitBricks:

```go
profitbricks.SetAuth("username", "password")
```

Set depth:

```go
profitbricks.SetDepth("5")
```

Depth controls the amount of data returned from the REST server ( range 1-5 ). The larger the number the more information is returned from the server. This is especially useful if you are looking for the information in the nested objects.

**Caution**: You will want to ensure you follow security best practices when using credentials within your code or stored in a file.

# How To's

## How To: Create Data Center

ProfitBricks introduces the concept of Data Centers. These are logically separated from one another and allow you to have a self-contained environment for all servers, volumes, networking, snapshots, and so forth. The goal is to give you the same experience as you would have if you were running your own physical data center.

The following code example shows you how to programmatically create a data center:

```go
dcrequest := profitbricks.Datacenter{
		Properties: profitbricks.DatacenterProperties{
			Name:        "example.go3",
			Description: "description",
			Location:    "us/lasdev",
		},
	}

datacenter := profitbricks.CreateDatacenter(dcrequest)
```

## How To: Create Data Center with Multiple Resources

To create a complex Data Center you would do this. As you can see, you can create quite a few of the objects you will need later all in one request.:

```go
datacenter := model.Datacenter{
		Properties: model.DatacenterProperties{
			Name: "composite test",
			Location:location,
		},
		Entities:model.DatacenterEntities{
			Servers: &model.Servers{
				Items:[]model.Server{
					model.Server{
						Properties: model.ServerProperties{
							Name : "server1",
							Ram: 2048,
							Cores: 1,
						},
						Entities:model.ServerEntities{
							Volumes: &model.AttachedVolumes{
								Items:[]model.Volume{
									model.Volume{
										Properties: model.VolumeProperties{
											Type_:"HDD",
											Size:10,
											Name:"volume1",
											Image:"1f46a4a3-3f47-11e6-91c6-52540005ab80",
											Bus:"VIRTIO",
											ImagePassword:"test1234",
											SshKeys: []string{"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCoLVLHON4BSK3D8L4H79aFo..."},
										},
									},
								},
							},
							Nics: &model.Nics{
								Items: []model.Nic{
									model.Nic{
										Properties: model.NicProperties{
											Name : "nic",
											Lan : "1",
										},
									},
								},
							},
						},
					},
				},
			},
		},
	}
	
dc := CompositeCreateDatacenter(datacenter)

```


## How To: Delete Data Center

You will want to exercise a bit of caution here. Removing a data center will destroy all objects contained within that data center -- servers, volumes, snapshots, and so on.

The code to remove a data center is as follows. This example assumes you want to remove previously data center:

```go
profitbricks.DeleteDatacenter(response.Id)
```

## How To: Create Server

The server create method has a list of required parameters followed by a hash of optional parameters. The optional parameters are specified within the "options" hash and the variable names match the [REST API](https://devops.profitbricks.com/api/rest/) parameters.

The following example shows you how to create a new server in the data center created above:

```go
req := profitbricks.Server{
 		Properties: profitbricks.ServerProperties{
 			Name:  "go01",
 			Ram:   1024,
 			Cores: 2,
 		},
}
server := CreateServer(datacenter.Id, req)
```

## How To: List Available Images

A list of disk and ISO images are available from ProfitBricks for immediate use. These can be easily viewed and selected. The following shows you how to get a list of images. This list represents both CDROM images and HDD images.

```go
images := profitbricks.ListImages()
```

This will return a [collection](#Collection) object

## How To: Create Storage Volume

ProfitBricks allows for the creation of multiple storage volumes that can be attached and detached as needed. It is useful to attach an image when creating a storage volume. The storage size is in gigabytes.

```go
volumerequest := profitbricks.Volume{
		Properties: profitbricks.VolumeProperties{
			Size:        1,
			Name:        "Volume Test",
			LicenceType: "LINUX",
			Type:        "HDD",
		},
}

storage := CreateVolume(datacenter.Id, volumerequest)
```

## How To: Update Cores and Memory

ProfitBricks allows users to dynamically update cores, memory, and disk independently of each other. This removes the restriction of needing to upgrade to the next size available size to receive an increase in memory. You can now simply increase the instances memory keeping your costs in-line with your resource needs.

Note: The memory parameter value must be a multiple of 256, e.g. 256, 512, 768, 1024, and so forth.

The following code illustrates how you can update cores and memory:

```go
serverupdaterequest := profitbricks.ServerProperties{
	Cores: 1,
	Ram:   256,
}

resp := PatchServer(datacenter.Id, server.Id, serverupdaterequest)
```

## How To: Attach or Detach Storage Volume

ProfitBricks allows for the creation of multiple storage volumes. You can detach and reattach these on the fly. This allows for various scenarios such as re-attaching a failed OS disk to another server for possible recovery or moving a volume to another location and spinning it up.

The following illustrates how you would attach and detach a volume and CDROM to/from a server:

```go
profitbricks.AttachVolume(datacenter.Id, server.Id, volume.Id)
profitbricks.AttachCdrom(datacenter.Id, server.Id, images.Items[0].Id)

profitbricks.DetachVolume(datacenter.Id, server.Id, volume.Id)
profitbricks.DetachCdrom(datacenter.Id, server.Id, images.Items[0].Id)
```

## How To: List Servers, Volumes, and Data Centers

Go SDK provides standard functions for retrieving a list of volumes, servers, and datacenters.

The following code illustrates how to pull these three list types:

```go
volumes := profitbricks.ListVolumes(datacenter.Id)

servers := profitbricks.ListServers(datacenter.Id)

datacenters := profitbricks.ListDatacenters()
```

## Reference

## Data Centers

Virtual Data Centers are the foundation of the ProfitBricks platform. Virtual Data Centers act as logical containers for all other objects you will be creating, e.g., servers. You can provision as many data centers as you want. Data centers have their own private network and are logically segmented from each other to create isolation.

#### type Datacenter

| Property Name | Type | Description |
|---|---|---|
| Id | String | Unique identifier of the object|
| Type_ | String | Type of the object as returned from the Cloud API|
| Href | String | URL to the object’s representation|
| Headers | *http.Header | Response headers|
| Response | string | Raw JSON response|
| StatusCode | int | Http response status code |
| Metadata | *Metadata | See [Metadata](#metadata) |
| Properties| DatacenterProperties  | See [DatacenterProperties](#datacenterproperties)|
| Entities  | DatacenterEntities    |See [DatacenterEntities](#datacenterentities)|

#### type Metadata

| Property Name |  Type | Description |
|---|---|---|
|CreatedDate     |time.Time|The date when the resource was created.|
|CreatedBy       |string|The user who created the resource.|
|Etag            |string|The etag for the request.|
|LastModifiedDate|time.Time|The last time the resource has been modified.|
|LastModifiedBy  |string|The user who last modified the resource.|
|State            |string|*AVAILABLE* There are no pending modification requests for this item; *BUSY* There is at least one modification request pending and all following requests will be queued; *INACTIVE* Resource has been de-provisioned.|

#### type DatacenterProperties

| Property Name |  Type | Description |
|---|---|---|
|Name        |string| Name of the data center|
|Description |string|Description of the data center|
|Location    |string|Location of the data center|
|Version     |int32 |The version of the data center|

The following table outlines the locations currently supported:

| ID | Country | City |
|---|---|---|
| us/las | United States | Las Vegas |
| de/fra | Germany | Frankfurt |
| de/fkb | Germany | Karlsruhe |

#### type DatacenterEntities

| Property Name |  Type | Description |
|---|---|---|
|Servers       |*Servers      |See [Servers](#servers)|
|Volumes       |*Volumes      |See [Volumes](#volumes)|
|Loadbalancers |*Loadbalancers|See [LoadBalancers](#loadbalancers)|
|Lans          |*Lans         |See [Lans](#lans)|

#### type Datacenters

| Property Name |  Type | Description |
|---|---|---|
| Id | String | Unique identifier of the object|
| Type_ | String | Type of the object as returned from the Cloud API|
| Href | String | URL to the object’s representation|
| Headers | *http.Header | Response headers|
| Response | string | Raw JSON response|
| StatusCode | int | Http response status code |
| Metadata | *Metadata | See [Metadata](#metadata) |
| Items | []Datacenter | Array of [Datacenters](#datacenter)|

### List Data Centers

#### func ListDatacenters() [Datacenters](#type-datacenters)

---

### Retrieve a Data Center

#### func GetDatacenter(dcid string) [Datacenter](#type-datacenter)

---

### Create a Data Center

#### func CreateDatacenter(dc Datacenter) [Datacenter](#type-datacenter)

Notes:

* _The value for `name` cannot contain the following characters: (@, /, , |, ‘’, ‘)._
* _You cannot change a data center's `location` once it has been provisioned._

---

### Update a Data Center

#### func PatchDatacenter(dcid string, obj [DatacenterProperties](#type-datacenterproperties)) [Datacenter](#type-datacenter)

---

### Delete a Data Center

#### func DeleteDatacenter(dcid string) [Resp](#type-resp)

**Note**: This is a highly destructive operation that will remove both the data center and **all** objects within the datacenter. This should be used with extreme caution.

## Servers

Description here...

#### type Server

| Name | Type | Description |
|---|---|---|
| Id | String | Unique identifier of the object|
| Type_ | String | Type of the object as returned from the Cloud API|
| Href | String | URL to the object’s representation|
| Headers | *http.Header | Response headers|
| Response | string | Raw JSON response|
| StatusCode | int | Http response status code |
| Id         |string|      |           
| Type_      |string| |
| Metadata | *Metadata | See [Metadata](#metadata) |
| Properties | ServerProperties | See [ServerProperties](#serverproperties)|
| Entities   | *ServerEntities | See [ServerEntities](#serverentities)|

The following table outlines the various licence types you can define:

| Licence Type | Description |
|---|---|
| WINDOWS | You must specify this if you are using your own, custom Windows image due to Microsoft's licensing terms. |
| LINUX | |
| UNKNOWN | If you are using an image uploaded to your account your OS Type will inherit as UNKNOWN. |

The following table outlines the availability zones currently supported:

| Availability Zone | Description |
|---|---|
| AUTO | Automatically selected zone |
| ZONE_1 | Zone 1 |
| ZONE_2 | Zone 2 |
| ZONE_3 | Zone 3 |
	
#### type ServerProperties

| Name | Type | Description |
|---|---|---|
| Name | string | The hostname of the server.|
| Cores | int | The total number of cores for the server.|
| Ram | int | The amount of memory for the server in MB, e.g. 2048. | 
| AvailabilityZone | string |The availability zone in which the server should exist.|
| VmState | string | Status of the virtual Machine.|
| BootCdrom | *ResourceReference | 	Reference to a CD-ROM used for booting. |
| BootVolume | *ResourceReference | Reference to a Volume used for booting.|
| CpuFamily | string | Type of CPU assigned. |

#### type ServerEntities

| Name | Type | Description |
|---|---|---|
|Cdroms  |*Cdroms | See [Cdrom](#cdroms) |
|Volumes |*Volumes | See [Volumes](#volumes)|
|Nics    |*Nics | See [Nics](#nics)|

#### type Servers

| Name | Type | Description |
|---|---|---|
| Id | String | Unique identifier of the object|
| Type_ | String | Type of the object as returned from the Cloud API|
| Href | String | URL to the object’s representation|
| Headers | *http.Header | Response headers|
| Response | string | Raw JSON response|
| StatusCode | int | Http response status code |
| Metadata | *Metadata | See [Metadata](#metadata) |
| Items | []Server | Array of [Servers](#server) |

### List Servers

#### func ListServers(dcid string) [Server](#type-server)

---

### Retrieve a Server

Returns information about a server such as its configuration, provisioning status, etc.

#### func GetServer(dcid, srvid string) [Servers](#type-servers)

---

### Create a Server

Creates a server within an existing data center. You can configure additional properties such as specifying a boot volume and connecting the server to an existing LAN.

#### func CreateServer(dcid string, server [Server](Server)) [Server](#type-server)

---

### Update a Server

Perform updates to attributes of a server.

#### func PatchServer(dcid string, srvid string, props [ServerProperties](#type-serverproperties)) [Server](#type-server)

---

### Delete a Server

This will remove a server from a data center. NOTE: This will not automatically remove the storage volume(s) attached to a server. A separate API call is required to perform that action.

The following table describes the request arguments:

| Name | Type | Description | Required |
|---|---|---|---|
| dcid | string | The ID of the data center. | Yes |
| srvid | string | The ID of the server. | Yes |

#### func DeleteServer(dcid, srvid string) [Resp](#type-resp)

---

### Reboot a Server

This will force a hard reboot of the server. Do not use this method if you want to gracefully reboot the machine. This is the equivalent of powering off the machine and turning it back on.

#### func RebootServer(dcid, srvid string) [Resp](#type-resp)

---

### Start a Server

This will start a server. If the server's public IP was deallocated then a new IP will be assigned.

#### func StartServer(dcid, srvid string) [Resp](#type-resp)

---

### Stop a Server

This will stop a server. The machine will be forcefully powered off, billing will cease, and the public IP, if one is allocated, will be deallocated.

#### func StopServer(dcid, srvid string) [Resp](#type-resp)

---

### Attach a CDROM

This will attach a CDROM to the server.

#### func AttachCdrom(dcid string, srvid string, cdid string) [Image](#type-image)

---

### Detach a CDROM

This will detach the CDROM from the server. Depending on the volume "hot_unplug" settings, this may result in the server being rebooted.

#### func DetachCdrom(dcid, srvid, cdid string) [Resp](#type-resp)

---

### List attached CDROMs

This will list CDROMs that are attached to the server

#### func ListAttachedCdroms(dcid, srvid string) [Images](#type-images)

---

### Get attached CDROM

This will retrieve a CDROM that is attached to the server.

#### func GetAttachedCdrom(dcid, srvid, cdid string) [Image](#type-image)

## Volumes

Description here...

#### type Volume

| Name | Type | Description |
|---|---|---|
| Id | String | Unique identifier of the object|
| Type_ | String | Type of the object as returned from the Cloud API|
| Href | String | URL to the object’s representation|
| Headers | *http.Header | Response headers|
| Response | string | Raw JSON response|
| StatusCode | int | Http response status code |
| Metadata | *Metadata | See [Metadata](#metadata) |
| Properties | VolumeProperties | See [VolumeProperties](#volueproperties) |

#### type VolumeProperties

| Name | Type | Description |
|---|---|---|
| Name                | string  |	The name of the volume. | 
| Type                | string  | The volume type, HDD or SSD. | 
| Size                | int     | The size of the volume in GB. | 
| AvailabilityZone    | string  | The storage availability zone assigned to the volume. | 
| Image               | string  | The image or snapshot ID.| 
| ImagePassword       | string  | Always returns "null".| 
| SshKeys             | []string| Always returns "null".| 
| Bus                 | string  | The bus type: VIRTIO or IDE. Returns "null" if not connected to a server.| 
| LicenceType         | string  | Licence type. | 
| CpuHotPlug          | bool    | This volume is capable of CPU hot plug | 
| CpuHotUnplug        | bool    | This volume is capable of CPU hot unplug | 
| RamHotPlug          | bool    | This volume is capable of memory hot plug | 
| RamHotUnplug        | bool    | This volume is capable of memory hot unplug | 
| NicHotPlug          | bool    | This volume is capable of nic hot plug | 
| NicHotUnplug        | bool    | This volume is capable of nic hot unplug | 
| DiscVirtioHotPlug   | bool    | This volume is capable of VirtIO drive hot plug | 
| DiscVirtioHotUnplug | bool    | This volume is capable of VirtIO drive hot unplug | 
| DiscScsiHotPlug     | bool    | This volume is capable of SCSI drive hot plug | 
| DiscScsiHotUnplug   | bool    | This volume is capable of SCSI drive hot unplug | 
| DeviceNumber        | int64   | The LUN ID of the storage volume. | 

#### type Volumes

| Name | Type | Description |
|---|---|---|
| Id | String | Unique identifier of the object|
| Type_ | String | Type of the object as returned from the Cloud API|
| Href | String | URL to the object’s representation|
| Headers | *http.Header | Response headers|
| Response | string | Raw JSON response|
| StatusCode | int | Http response status code |
| Metadata | *Metadata | See [Metadata](#metadata) |
| Items | []Volume | Array of [Volumes](#server) |

### List Volumes

#### func ListVolumes(dcid string) [Volumes](#type-volumes)

---

### Retrieve a Volume

#### func GetVolume(dcid string, volumeId string) [Volume](#type-volume)

---

### Update a Volume

#### func PatchVolume(dcid string, volid string, request VolumeProperties) [Volume] (#type-volume)

---

### Create a Volume

#### func CreateVolume(dcid string, request Volume) [Volume](#type-volume)

---

### Delete a Volume

This will delete a volume.

#### func DeleteVolume(dcid, volid string) [Resp](#type-resp)

---

## NICs

### type Nic

| Name | Type | Description |
|---|---|---|
| Id | String | Unique identifier of the object|
| Type_ | String | Type of the object as returned from the Cloud API|
| Href | String | URL to the object’s representation|
| Headers | *http.Header | Response headers|
| Response | string | Raw JSON response|
| StatusCode | int | Http response status code |
| Metadata | *Metadata | See [Metadata](#metadata) |
| Properties |NicProperties | See [NicProperties](#nicproperties) |
| Entities |*NicEntities | See [NicEntities](#nicentities)|

### type NicProperties 

| Name | Type | Description |
|---|---|---|
| Name           | string   |The name of the NIC.|
| Mac            | string   |The MAC address of the NIC.|
| Ips            | []string |IPs assigned to the NIC represented as a collection of strings|
| Dhcp           | bool     |Boolean value that indicates if the NIC is using DHCP or not.|
| Lan            | int      |The LAN ID the NIC sits on.|
| FirewallActive | bool     |A true value indicates the firewall is enabled. A false value indicates the firewall is disabled.|
| Nat            | bool     |Boolean value indicating if the private IP address has outbound access to the public internet.|

### type NicEntities

| Name | Type | Description |
|---|---|---|
|Firewallrules | talk*Firewallrules | See [Firewallrules](#firewallrules)|

### type Nics 

| Name | Type | Description |
|---|---|---|
| Id | String | Unique identifier of the object|
| Type_ | String | Type of the object as returned from the Cloud API|
| Href | String | URL to the object’s representation|
| Headers | *http.Header | Response headers|
| Response | string | Raw JSON response|
| StatusCode | int | Http response status code |
| Metadata | *Metadata | See [Metadata](#metadata) |
| Items | []Nic | Array of [Nics](#nic) |

### List NICs

Retrieve a list of LANs within the data center.

#### func ListNics(dcid, srvid string)[Nics](#nics)

---

#### Get a NIC

Retrieves the attributes of a given NIC.

#### func GetNic(dcid, srvid, nicid string) [Nic](#nic)

---

### Create a NIC

Adds a NIC to the target server.

#### func CreateNic(dcid string, srvid string, request Nic) [Nic](#nic)

---

### Update a NIC

You can update -- in full or partially -- various attributes on the NIC; however, some restrictions are in place:

The primary address of a NIC connected to a load balancer can only be changed by changing the IP of the load balancer. You can also add additional reserved, public IPs to the NIC.

The user can specify and assign private IPs manually. Valid IP addresses for private networks are 10.0.0.0/8, 172.16.0.0/12 or 192.168.0.0/16.

#### func PatchNic(dcid string, srvid string, nicid string, obj [NicProperties](#type-nicproperties)) [Nic](#type-nic)

---

### Delete a NIC

This will delete a volume

#### func DeleteNic(dcid, srvid, nicid string) [Resp](#resp)

---

## Example

```go
package main

import (
	"fmt"
	"time"

	"github.com/profitbricks/profitbricks-sdk-go"
)

func main() {

	//Sets username and password
	profitbricks.SetAuth("username", "password")
	//Sets depth.
	profitbricks.SetDepth("5")

	dcrequest := profitbricks.Datacenter{
		Properties: profitbricks.DatacenterProperties{
			Name:        "example.go3",
			Description: "description",
			Location:    "us/lasdev",
		},
	}

	datacenter := profitbricks.CreateDatacenter(dcrequest)

	serverrequest := profitbricks.Server{
		Properties: profitbricks.ServerProperties{
			Name:  "go01",
			Ram:   1024,
			Cores: 2,
		},
	}
	server := profitbricks.CreateServer(datacenter.Id, serverrequest)

	volumerequest := profitbricks.Volume{
		Properties: profitbricks.VolumeProperties{
			Size:        1,
			Name:        "Volume Test",
			LicenceType: "LINUX",
			Type:        "HDD",
		},
	}

	storage := profitbricks.CreateVolume(datacenter.Id, volumerequest)

	serverupdaterequest := profitbricks.ServerProperties{
		Name:  "go01renamed",
		Cores: 1,
		Ram:   256,
	}

	profitbricks.PatchServer(datacenter.Id, server.Id, serverupdaterequest)
	//It takes a moment for a volume to be provisioned so we wait.
	time.Sleep(60 * time.Second)

	profitbricks.AttachVolume(datacenter.Id, server.Id, storage.Id)

	volumes := profitbricks.ListVolumes(datacenter.Id)
	fmt.Println(volumes.Items)
	servers := profitbricks.ListServers(datacenter.Id)
	fmt.Println(servers.Items)
	datacenters := profitbricks.ListDatacenters()
	fmt.Println(datacenters.Items)

	profitbricks.DeleteServer(datacenter.Id, server.Id)
	profitbricks.DeleteDatacenter(datacenter.Id)
}
```

# Support
You are welcome to contact us with questions or comments at [ProfitBricks DevOps Central](https://devops.profitbricks.com/). Please report any issues via [GitHub's issue tracker](https://github.com/profitbricks/profitbricks-sdk-go/issues).
