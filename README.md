import java.util.Calendar;
import java.util.Date;

interface BasicAccount
{
	String getName();

	double getBalance();
}

interface DepositableAccount extends BasicAccount
{
	double deposit(double amount) throws Exception;
}

interface WithdrawableAccount extends BasicAccount
{
	double withdraw(double amount) throws Exception;
}

interface InterestableAccount extends BasicAccount
{
	double compute_interest() throws Exception;
}

interface AllFunctionAccount extends DepositableAccount,WithdrawableAccount,InterestableAccount
{

}

public abstract class Account implements AllFunctionAccount
{
	protected String accountType;
	protected String accountName;
	protected double accountBalance;
	protected double accountInterestRate;
	protected Date openDate;
	protected Date lastIntersetDate;
	protected Date lastTransactionDate;

	public String getName()
	{
		return (accountName);
	}
	public double getBalance()
	{
		return (accountBalance);
	}

	abstract double deposit(double amount,Date depositDate) throws Exception;

	public double deposit(double amount) throws Exception
	{
		Date depositDate = new Date();
		return (deposit(amount,depositDate));
	}

	abstract double withdraw(double amount,Date withdrawDate) throws Exception;

	public double withdraw(double amount) throws Exception
	{
		Date withdrawDate = new Date();
		return (withdraw(amount,withdrawDate));
	}

	abstract double compute_interest(Date interestDate) throws Exception;

	public double compute_interest() throws Exception
	{
		Date interestDate = new Date();
		return (compute_interest(interestDate));
	}
}

/*	
*	Derived class : CheckingAccount
*
*	Description :
*		interest is computed daily;
*		there's no fee for withdrawals;
*		there is a minimum balance of $1000.
*/
class CheckingAccount extends Account 
{
	CheckingAccount(String name,double initDeposit)
	{
		accountType = "CheckingAccount";
		accountName = name;
		accountBalance = initDeposit;
		accountInterestRate = 0.001;
		lastTransactionDate = lastIntersetDate = openDate = new Date();
	}

	CheckingAccount(String name,double initDeposit,Date firstDate)
	{
		accountType = "CheckingAccount";
		accountName = name;
		accountBalance = initDeposit;
		accountInterestRate = 0.001;
		lastTransactionDate = lastIntersetDate = openDate = firstDate;
	}

	public double deposit(double amount, Date depositDate) throws Exception
	{
		lastTransactionDate = new Date();
		accountBalance += amount;
		return (accountBalance);
	}

	public double withdraw(double amount, Date withdrawDate) throws Exception
	{
		if((accountBalance - amount) < 1000)
		{
			throw new Exception("Not Enough Balance to withdraw from : " + accountName);
		}
		else
		{
			lastTransactionDate = new Date();
			accountBalance -= amount;
			System.out.println("Transaction Fee : $" + 0.0);
			return (accountBalance);
		}
	}
	public double compute_interest(Date interestDate) throws Exception
	{
		if(interestDate.before(lastIntersetDate))
		{
			throw new Exception("Illegal time to compute interest for :" + accountName);
		}
		int numberOFdays = (int)((interestDate.getTime() - lastIntersetDate.getTime())/(24*3600*1000.0));
		System.out.println("Number of days since last interest date is " + numberOFdays);
		double profit = (double)numberOFdays / 365.0 * accountInterestRate * accountBalance ;
		System.out.println("Interest earned : " + (profit = Math.floor(profit)));
		lastIntersetDate = interestDate;
		accountBalance += profit;
		return (accountBalance);
	}
}

/*
*	Derived class : SavingAccount
*
*	Description :
*		monthly interest;
*		fee of $1 for every transaction, except the first three per month are free;
*		no minimum balance.
*/
class SavingAccount extends Account 
{
	double transactionFee;
	int transTimeThisMonth = 0;

	SavingAccount(String name, double initDeposit)
	{
		accountType = "SavingAccount";
		accountName = name;
		accountBalance = initDeposit;
		accountInterestRate = 0.3;
		lastIntersetDate = lastTransactionDate = openDate = new Date();
	}

	SavingAccount(String name, double initDeposit, Date firstDate)
	{
		accountType = "SavingAccount";
		accountName = name;
		accountBalance = initDeposit;
		accountInterestRate = 0.3;
		lastIntersetDate = lastTransactionDate = openDate = firstDate;
	}

	public boolean IsThisMonthHaveTrans(Date transactionDate) //判斷該月有無交易
	{
		System.out.println("Last withdraw time : " + lastIntersetDate);
		System.out.println("This withdraw time : " + transactionDate);
		if(lastTransactionDate.getYear() == transactionDate.getYear() &&
			lastIntersetDate.getMonth() == transactionDate.getMonth())
			return true;
		return false;
	}

	public double deposit(double amount, Date depositDate) throws Exception
	{
		transactionFee = 0;
		if(!IsThisMonthHaveTrans(depositDate))
		{
			transTimeThisMonth = 0;
		}
		else if(transTimeThisMonth > 2)
		{
			transactionFee = 1;
		}
		accountBalance += amount - transactionFee;
		System.out.println("Transaction Fee : $" + transactionFee);
		lastTransactionDate = depositDate;
		transTimeThisMonth++;
		System.out.println("This month totally transaction time : " + transTimeThisMonth);
		return (accountBalance);
	}

	public double withdraw(double amount, Date withdrawDate) throws Exception
	{
		transactionFee = 0;
		if(!IsThisMonthHaveTrans(withdrawDate))
			transTimeThisMonth = 0;
		else if(transTimeThisMonth > 2)
			transactionFee = 1;
		if((accountBalance - amount - transactionFee) < 0)
		{
			throw new Exception("Not Enough Balance to withdraw from : " + accountName);
		}
		else
		{
			accountBalance -= amount + transactionFee;
			lastTransactionDate = withdrawDate;
			transTimeThisMonth++;
			System.out.println("This month totally transaction time : " + transTimeThisMonth);
			return (accountBalance);
		}
	}

	public double compute_interest(Date interestDate) throws Exception
	{
		if(interestDate.before(lastIntersetDate))
		{
			throw new Exception("Illegal time to compute interest for :" + accountName);
		}
		int numberOFmonths = (interestDate.getYear() - lastIntersetDate.getYear())*12 + interestDate.getMonth() - lastIntersetDate.getMonth();
		System.out.println("Number of months since last interest date is " + numberOFmonths);
		double profit = (double)numberOFmonths / 12.0 * accountInterestRate * accountBalance ;
		System.out.println("Interest earned : " + (profit = Math.floor(profit)));
		lastIntersetDate = interestDate;
		accountBalance += profit;
		return (accountBalance);
	}
}

/*
*	Derived class : CDAccount
*
*	Description :
*		monthly interest;
*		fixed amount and duration;
*		-During the duration: can't deposit anything , withdrawals cost a  $250 fee.
*		-After the duration: interest payments stop , withdraw w/o fee.
*/		
class CDAccount extends Account 
{
	Calendar expireDate;

	CDAccount(String name, double initDeposit)
	{
		accountType = "CDAccount";
		accountName = name;
		accountBalance = initDeposit;
		accountInterestRate = 0.002;
		lastTransactionDate = lastIntersetDate = openDate = new Date();
		expireDate = Calendar.getInstance();
		expireDate.setTime(lastIntersetDate);
		expireDate.add(Calendar.YEAR,1);
	}

	CDAccount(String name, double initDeposit, Date firstDate)
	{
		accountType = "CDAccount";
		accountName = name;
		accountBalance = initDeposit;
		accountInterestRate = 0.002;
		lastTransactionDate = lastIntersetDate = openDate = firstDate;
		expireDate = Calendar.getInstance();
		expireDate.setTime(lastIntersetDate);
		expireDate.add(Calendar.YEAR,1);
	}

	public double deposit(double amount, Date depositDate) throws Exception
	{
		if(depositDate.before(expireDate.getTime()))
		{
			throw new Exception("You can't deposit during this time!");
		}
		lastTransactionDate = new Date();
		accountBalance += amount;
		return (accountBalance);
	}

	public double withdraw(double amount, Date withdrawDate) throws Exception
	{
		if(withdrawDate.before(expireDate.getTime()))
		{
			if((accountBalance - amount - 250.0) < 0)
			{
				throw new Exception("Not Enough Balance to withdraw from : " + accountName);
			}
			else
			{
				lastTransactionDate = new Date();
				accountBalance -= amount + 250.0;
				System.out.println("Transaction Fee : $" + 250.0);
				return (accountBalance);
			}
		}
		else
		{
			if((accountBalance - amount) < 0)
			{
				throw new Exception("Not Enough Balance to withdraw from : " + accountName);
			}
			else
			{
				lastTransactionDate = new Date();
				accountBalance -= amount;
				System.out.println("Transaction Fee : $" + 0.0);
				return (accountBalance);
			}
		}
	}

	public double compute_interest(Date interestDate) throws Exception
	{
		if(interestDate.after(expireDate.getTime()))
		{
			throw new Exception("Illegal time to compute interest for : " + accountName);
		}
		int numberOFmonths = (interestDate.getYear() - lastIntersetDate.getYear()) * 12 + (interestDate.getMonth() - lastIntersetDate.getMonth());
		System.out.println("Number of months since last interest date is " + numberOFmonths);
		double profit = (accountInterestRate / numberOFmonths) * accountBalance;
		System.out.println("Interest earned : " + (profit = Math.floor(profit)));
		lastIntersetDate = interestDate;
		accountBalance += profit;
		return (accountBalance);
	}
}

/*
*	Derived class : LoanAccount
*
*	Description : like a saving account
*		"negative" balance is allow.
*		you can't withdraw when balance is negative,
*		but of course you can deposit.
*/
class LoanAccount extends SavingAccount
{
	LoanAccount(String name, double initDeposit)
	{
		super(name, initDeposit);
		accountType = "LoanAccount";
	}

	LoanAccount(String name, double initDeposit, Date firstDate)
	{
		super(name, initDeposit, firstDate);
		accountType = "LoanAccount";
	}

	public double withdraw(double amount, Date withdrawDate) throws Exception
	{
		if(accountBalance < 0)
		{
			throw new Exception("Not Enough Balance to withdraw from : " + accountName);
		}
		else
		{
			lastTransactionDate = new Date();
			accountBalance -= amount;
			return (accountBalance);
		}
	}
}
