1|Jam.py demo| 
def on_created(task):
  pass
5|Reports|# import os
# from subprocess import Popen, STDOUT, PIPE

# def on_convert_report(report):
#     try:
#         if os.name == "nt":
#             import _winreg
#             regpath = "SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\App Paths\\soffice.exe"
#             root = _winreg.OpenKey(_winreg.HKEY_LOCAL_MACHINE, regpath)
#             s_office = _winreg.QueryValue(root, "")
#         else:
#             s_office = "soffice"
#         convertion = Popen([s_office, '--headless', '--convert-to', report.ext,
#             report.report_filename, '--outdir', os.path.join(report.task.work_dir, 'static', 'reports') ],
#             stderr=STDOUT,stdout=PIPE)#, shell=True)
#         out, err = convertion.communicate()
#         converted = True
#     except Exception as e:
#         print(e)

13|Genres|

def on_apply(item, delta, params):
	if not delta.id.value:
	    delta.edit()
#	    delta.id.value = //here must be some function or request that returns new id value
	    delta.post()
15|Tracks|
def set_media_type(item, media_type, selections):
    copy = item.copy()
    copy.set_where(id__in=selections)
    copy.open(fields=['id', 'media_type'])
    for c in copy:
        c.edit()
        c.media_type.value = media_type
        c.post()
    c.apply()
    return True
16|Invoices|
def on_apply(item, delta, params, connection): 
    tracks = item.task.tracks.copy(handlers=False)
    changes = {}
    delta.update_deleted()    
    for d in delta:
        for t in d.invoice_table:
            if not changes.get(t.track.value):
                changes[t.track.value] = 0
            if t.rec_inserted():
                changes[t.track.value] += t.quantity.value
            elif t.rec_deleted():
                changes[t.track.value] -= t.quantity.value                
            elif t.rec_modified():
                changes[t.track.value] += t.quantity.value - t.quantity.old_value
    ids = list(changes.keys())
    tracks.set_where(id__in=ids)                
    tracks.open()
    for t in tracks:
        q = changes.get(t.id.value)
        if q:
            t.edit()
            t.tracks_sold.value += q
            t.post()
    tracks.apply(connection)

19|Print invoice|# -*- coding: utf-8 -*-

def on_generate(report):
    invoices = report.task.invoices.copy()
    invoices.set_where(id=report.id.value)
    invoices.open()

    customer = invoices.firstname.display_text + ' ' + invoices.customer.display_text
    address = invoices.billing_address.display_text
    city = invoices.billing_city.display_text + ' ' + invoices.billing_state.display_text + ' ' + \
        invoices.billing_country.display_text
    date = invoices.invoice_date.display_text
    shipped = invoices.billing_address.display_text + ' ' + invoices.billing_city.display_text + ' ' + \
        invoices.billing_state.display_text + ' ' + invoices.billing_country.display_text
    taxrate = invoices.taxrate.display_text
    report.print_band('title', locals())

    tracks = invoices.invoice_table
    tracks.open()
    for t in tracks:
        quantity = t.quantity.display_text
        track = t.track.display_text
        if t.album.display_text:
            track += ', album: ' + t.album.display_text
        if t.artist.display_text:
            track += ', artist: ' + t.artist.display_text
        unitprice = t.unitprice.display_text
        sum = t.amount.display_text
        report.print_band('detail', locals())

    subtotal = invoices.subtotal.display_text
    tax = invoices.tax.display_text
    total = invoices.total.display_text
    report.print_band('summary', locals())
20|Customer purchases |# -*- coding: utf-8 -*-

from jam.common import cur_to_str

def on_generate(report):
    range = '%s - %s' % (report.invoicedate1.display_text, report.invoicedate2.display_text)
    report.print_band('title', locals())
    inv = report.task.invoices.copy()    
    if report.customer.value and len(report.customer.value) == 1:
        customer = report.customer.value[0]
        cust = report.task.customers.copy()
        cust.set_where(id=customer)
        cust.open()
        name = cust.firstname.display_text + ' ' + cust.lastname.display_text
        address = cust.address.display_text
        report.print_band('customer', locals())
        report.print_band('cust_header')
        inv.set_where(customer=customer, invoice_date__ge=report.invoicedate1.value,
            invoice_date__le=report.invoicedate2.value)
        inv.set_order_by('invoice_date')
        inv.open()
        total = 0
        for i in inv:
            date = i.invoice_date.display_text
            subtotal = i.subtotal.display_text
            tax = i.tax.display_text
            sum = i.total.display_text
            total += i.total.value
            report.print_band('cust_detail', locals())
        total = cur_to_str(total)
        report.print_band('cust_summary', locals())
    else:
        report.print_band('header')
        inv.set_where(invoice_date__ge=report.invoicedate1.value,
            invoice_date__le=report.invoicedate2.value, customer__in=report.customer.value)
        inv.open(fields=['customer', 'firstname', 'id', 'total'], 
            funcs={'id': 'count', 'total': 'sum', 'firstname': 'max'}, 
            group_by=['customer'], order_by=['customer'])
        total = 0
        for i in inv:
            name = i.firstname.display_text + ' ' + i.customer.display_text
            count = i.id.value
            sum = i.total.display_text
            total += i.total.value
            report.print_band('detail', locals())
        total = cur_to_str(total)
        report.print_band('summary', locals())

22|Customer list|
def on_generate(report):
    cust = report.task.customers.copy()
    if report.customers.value:
        cust.set_where({'id__in': report.customers.value})
    cust.set_order_by('lastname')
    cust.open()

    report.print_band('title')

    for c in cust:
        firstname = c.firstname.display_text
        lastname = c.lastname.display_text
        company = c.company.display_text
        country = c.country.display_text
        address = c.address.display_text
        phone = c.phone.display_text
        email = c.email.display_text
        report.print_band('detail', locals())

# def on_parsed(report):
#     report.hide_columns(['A', 'B'])
25|Mail|
import smtplib

def send_email(item, selected, subject, mess):
    if not item.can_create():
        raise Exception('You are not allowed to send emails.')
    cust = item.task.customers.copy()
    cust.set_where(id__in=selected)
    cust.open()
    to = []
    for c in cust:
        to.append(c.email.value)

        
    raise Exception('This is a demo app. <br> For a real application, ' +
        u'you must change the code in the server module of the mail item.')        
        
    # to send email using gmail you must turn on "Access for less secure apps"
    # https://www.google.com/settings/security/lesssecureapps
    
    # #set gmail_user and gmail_pwd to your email and password
    
    
    gmail_user = 'jam-py-demo@gmail.com'
    gmail_pwd = 'pwd'

    message = 'From: %s\nTo: %s\nSubject: %s\n\n%s' % (gmail_user, ", ".join(to), subject, mess)    
    server = smtplib.SMTP("smtp.gmail.com", 587)
    server.ehlo()
    server.starttls()
    server.login(gmail_user, gmail_pwd)
    server.sendmail(gmail_user, to, message)
    server.close()


