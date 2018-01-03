 OpenERP在哪储存附件？

我们知道对OpenERP中的每个内部对象（比如：业务伙伴，采购订单，销售订单，发货单，等等）我们都可以添加任意的附件，如图片，文档，视频等。那么这些附件在OpenERP内部是如何管理的呢？

默认情况下，这些附件在OpenERP v7中是保存在数据库中的。我们知道当附件的数量比较大时，这会严重影响数据库的性能。其实在OpenERP 中我们可以通过设置ir.config.parameter参数来使附件保存在文件系统中，具体菜单位置是：”设置-技术-参数-系统参数-ir_attachement.location” (Settings->Technical->Parameters-System parameters- ir_attachment.location）

比如我们将ir_attachment.location设置为file:///filestore

那么这些附件就会保存在openerp根目录/filestore下, 系统使用sha1哈希算法来创建文件名所以重复的文件在系统中并不会多占空间。

目前只支持file:///协议，实际上我们可以很容易通过扩增模块来支持比如amazons3:///协议，这样我们就可以将附件保存在亚马逊的S3云服务了。

数据库保存附件的模式下，数据是保存在ir_attachment.db_datas中
文件系统保存附件的模式下，文件名保存在ir_attachment.db_datas_fname中

我们尚为提供这两种模式的自动转换机制。所以，如果你设置了这个参数，那么已存在的附件仍将保存在数据库中，只有新附件会保存在文件系统中，系统会尝试访问这两个不同的位置，所以也没什么问题（先检查db_datas,然后再检查db_datas_fname)

注：本文末尾提供的脚本可以自动将现有数据库中的附件转换到文件系统中

如果你移除了这个参数，你需要设法将在文件系统中保存的附件存回到数据库中，因为系统就只会通过数据库来检查附件了。

将现有数据库中的附件数据转移到文件系统中的脚本(替换URL为您的OpenERP实际访问URL地址）：
#!/usr/bin/python
 
import xmlrpclib
 
username = 'admin' #the user
pwd = 'password'      #the password of the user
dbname = 'database'    #the database
 
# Get the uid
sock_common = xmlrpclib.ServerProxy ('<URL>/xmlrpc/common')
uid = sock_common.login(dbname, username, pwd)
sock = xmlrpclib.ServerProxy('<URL>/xmlrpc/object')
 
def migrate_attachment(att_id):
    # 1. get data
    att = sock.execute(dbname, uid, pwd, 'ir.attachment', 'read', att_id, ['datas'])
 
    data = att['datas']
 
    # Re-Write attachment
    a = sock.execute(dbname, uid, pwd, 'ir.attachment', 'write', [att_id], {'datas': data})
 
# SELECT attachments:
att_ids = sock.execute(dbname, uid, pwd, 'ir.attachment', 'search', [('store_fname','=',False)])
 
cnt = len(att_ids)
i = 0
for id in att_ids:
    att = sock.execute(dbname, uid, pwd, 'ir.attachment', 'read', id, ['datas','parent_id'])
 
    migrate_attachment(id)
    print 'Migrated ID %d (attachment %d of %d)' % (id,i,cnt)
    i = i + 1
 
print "done ..."

运行这个脚本后，我们还需要清除ir_attachements表:
update ir_attachment set db_datas = null where store_fname is not null
vacuum (full, analyze) ir_attachment
 
【原地址：http://cn.openerp.cn/where_to_store_attachement_in_openerp_7】
