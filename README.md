# E-Shop-Management
import IPython.display
import pandas as pd
import mysql.connector
mydb=mysql.connector.connect(host='localhost',user='root',passwd='1234',database='Eshop')

def admin_menu():
    ch5='y'
    while ch5=='y' or ch5=='Y':
        print(" 1. Add new product\n 2. Add stock to existing\n 3. View all product\n 4. Modify product\n 5. Delete data\n 6. Exit")
        ch1=int(input("Enter your choice"))
        if(ch1==1):
            add_new()
        elif(ch1==2):
            add_stock()
        elif(ch1==3):
            view_all()
        elif(ch1==4):
            modify()
        elif(ch1==5):
            delete()
        elif(ch1==6):
            return
        else:
            print("Invalid Input")
        ch5=input("Do you want to continue? ")

def add_new():
    pid=input("Enter Product ID: ")
    pname=input("Enter Product Name: ")
    qty=int(input("Enter Quantity: "))
    price=float(input("Enter Price: "))
    disc=int(input("Enter discount: "))
    cursor1=mydb.cursor()
    cursor1.execute("Insert into items values ('"+pid+"','"+pname+"','"+str(qty)+"','"+str(price)+"','"+str(disc)+"')")
    mydb.commit()

def add_stock():
    pid=input("Enter Product ID to add stock: ")
    qty1=int(input("Enter quantity to add: "))
    cursor1=mydb.cursor()
    cursor1.execute("select qty from items where PId='"+pid+"'")
    n=cursor1.fetchone()[0] # so that it can execute only values inside the table
    qty1=qty1+n
    cursor1.execute("Update items set qty='"+str(qty1)+"' where PId='"+pid+"'")
    mydb.commit()

def view_all():
    cursor1=mydb.cursor()
    cursor1.execute("select * from items")
    data=list(cursor1.fetchall())
    k=pd.DataFrame(data,columns=['PId','PName','Qty','Price','Disc'])
    print(k.to_string(index=False))
    
def modify():
    print(" Modify:\n 1. Product Name\n 2. nPrice\n 3. Discount")
    cursor1=mydb.cursor()
    ch=int(input("Enter what you want to modify: "))
    if(ch==1):
        pid=input("Enter Product ID: ")
        pname=input("Enter Product name: ")
        cursor1.execute("Update items set PName='"+pname+"' where PId='"+pid+"'")
    elif(ch==2):
        pid=input("Enter Product ID: ")
        price=float(input("Enter Price: "))
        cursor1.execute("Update items set Price='"+str(price)+"' where PId='"+pid+"'")
    elif(ch==3):
        pid=input("Enter Product ID: ")
        disc=input("Enter Discount: ")
        cursor1.execute("Update items set Disc='"+str(disc)+"' where PId='"+pid+"'")
    else:
        print("Invalid Option.......")
    mydb.commit()

def delete():
    cursor1=mydb.cursor()
    pid=input("Enter Product ID you want to delete: ")
    cursor1.execute("delete from items where PId='"+pid+"'")
    mydb.commit()





def buyer_menu():
    print(" 1. View & Buy Products\n 2. Exit")
    ch2=int(input("Enter your choice: "))
    if(ch2==1):
        view_buy()
    elif(ch2==2):
        return
    else:
        print("Invalid Option.......")


def view_buy():
    view_all()
    revert={}
    ch3='y'
    while(ch3=='y' or ch3=="Y"):
        pid=input("Enter Product ID to buy: ")
        qty2=int(input("Enter quantity of product: "))
        cursor1=mydb.cursor()
        revert [pid]=qty2
        cursor1.execute("select qty from items where PId='"+pid+"'")
        n=int(cursor1.fetchone()[0])
        if(qty2<=n):
            cursor1.execute("select pname from items where PId='"+pid+"'")
            pname=cursor1.fetchone()[0]
            cursor1.execute("select price from items where PId='"+pid+"'")
            price=cursor1.fetchone()[0]
            cursor1.execute("select disc from items where PId='"+pid+"'")
            disc=cursor1.fetchone()[0]
            net_price=(qty2*price)-((disc*qty2*price)/100)
            cursor2=mydb.cursor()
            cursor2.execute("insert into bill values ('"+pid+"','"+pname+"','"+str(qty2)+"','"+str(price)+"','"+str(disc)+"','"+str(net_price)+"')")
            n=n-qty2
            cursor1.execute("update items set qty='"+str(n)+"' where pid='"+pid+"'")
            mydb.commit()
        else:
            print("Required quantity not available.......")
        ch3=input("Do you want to continue? ")
    print("Generating your bill...")
    cursor2.execute("select * from bill")
    bill=cursor2.fetchall()
    k=pd.DataFrame(bill,columns=['PId','PName','Qty','Price','Disc','Net_Price'])
    print(k.to_string(index=False))
    cursor2.execute("select sum(net_price) from bill")
    total=cursor2.fetchone()[0]
    print("Total payable amount is : Rs. ",total," only")
    ch7=input("Are you sure you want to purchase & pay amount?")
    if(ch7=='y' or ch7=='Y'):
        cursor2.execute("delete from bill")
        mydb.commit()
        print("Thanx for shoping with us, have a good day!")
    else:
        print("Cancelling your purchase, see you again soon!")
        for i in revert:
            a=i
            b=revert[str(a)]
            cursor1=mydb.cursor()
            cursor1.execute("select qty from items where pid='"+a+"'")
            qty=cursor1.fetchone()[0]
            qty=qty+b
            cursor1.execute("update items set qty='"+str(qty)+"' where pid='"+a+"'")
            mydb.commit()
        cursor2.execute("delete from bill")
        mydb.commit()
        
    
   
    



print("*******Welcome to XYZ Electronic Shop*******")
ch6='y'
while ch6=='y' or ch6=='Y':
    x=int(input("Enter 1 for admin or any other key for purchase: "))
    if(x==1):
        pwd=input("Enter your password: ")
        if(pwd=="admin"):
            print("Welcome Admin.......")
            admin_menu()
        else:
            print("Invalid Password.........try Again")
    else:
        print("Welcome to XYZ Electronic Shop")
        buyer_menu()
    ch6=input("Press 'y' to continue or any other key to exit")

