---
Title: Unit of Work στην C#: Η ιστορία της συνέπειας και της καθαρότητας στη διαχείριση δεδομένων
Description: 
Author: nikosst
Date: 2026-01-14T20:41:05.000Z
Robots: noindex,nofollow
Template: index
---
<p>Η διαχείριση δεδομένων σε εφαρμογές με βάσεις δεδομένων ήταν πάντα μια πρόκληση για τους προγραμματιστές. Πριν την ευρεία υιοθέτηση του Unit of Work, κάθε αλλαγή σε ένα αντικείμενο έπρεπε να εφαρμοστεί ξεχωριστά στη βάση, συχνά με πολλαπλά SaveChanges() calls. Αυτό δημιουργούσε κίνδυνο ασυνέπειας, πολλαπλές κλήσεις στη βάση και σύνθετη λογική rollback.</p>




<p><strong>Η εποχή πριν το Unit of Work</strong></p>

<p>Στις πρώτες εφαρμογές με ORM, η λογική ήταν απλή αλλά επικίνδυνη:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>using (var context = new AppDbContext())
{
    var product = context.Products.Find(1);
    product.Price = 100;
    context.SaveChanges(); // Άμεσο commit

    var log = new Log { Message = "Αλλάχθηκε τιμή προϊόντος" };
    context.Logs.Add(log);
    context.SaveChanges(); // Ξεχωριστό commit
}

</code></pre>

</div>



<p>Εδώ, κάθε call σε SaveChanges() δημιουργούσε ξεχωριστή συναλλαγή. Αν κάτι πήγαινε στραβά μετά την πρώτη αλλαγή, η εφαρμογή βρισκόταν σε κατάσταση ασυνέπειας. Ακόμη και με repositories, το commit ήταν συνήθως ξεχωριστό για κάθε repository, με αποτέλεσμα διάσπαρτη λογική και μεγαλύτερη πιθανότητα σφαλμάτων.</p>




<p><strong>Η γέννηση του Unit of Work</strong></p>

<p>Η ανάγκη ήταν προφανής: οι αλλαγές θα έπρεπε να συγκεντρώνονται σε ένα ενιαίο έργο (unit of work) και να εκτελούνται όλες μαζί ή καμία.</p>

<p>Το Unit of Work λειτουργεί ως διαμεσολαβητής μεταξύ της εφαρμογής και της βάσης δεδομένων. Παρακολουθεί όλα τα αντικείμενα που τροποποιούνται και εφαρμόζει τις αλλαγές με ένα ενιαίο commit, διασφαλίζοντας atomicity και συνέπεια.</p>




<p><strong>Τι κάνει και γιατί είναι σημαντικό</strong></p>

<ul>
<li>Παρακολούθηση αλλαγών: γνωρίζει ποια αντικείμενα έχουν τροποποιηθεί.</li>
<li>Atomic commit: όλες οι αλλαγές εφαρμόζονται ή απορρίπτονται μαζί.</li>
<li>Μείωση calls στη βάση: αντί για πολλαπλά SaveChanges(), έχουμε ένα μόνο.</li>
<li>Καθαρός κώδικας: τα repositories δεν χρειάζεται να ξέρουν πότε γίνεται commit.</li>
<li>Ευκολία στις δοκιμές: η λογική μπορεί να δοκιμαστεί χωρίς άμεση πρόσβαση στη βάση.</li>
</ul>




<p><strong>Παράδειγμα Unit of Work σε C#</strong><br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>using (var unitOfWork = new UnitOfWork(new AppDbContext()))
{
    var product = unitOfWork.Products.GetById(1);
    product.Price = 99.99m;

    var customer = new Customer { Name = "Γιάννης" };
    unitOfWork.Customers.Add(customer);

    unitOfWork.Complete(); // Όλες οι αλλαγές γίνονται commit μαζί
}

</code></pre>

</div>



<p>Αυτό εξασφαλίζει ότι όλες οι αλλαγές εκτελούνται ως ένα ενιαίο έργο, μειώνοντας τα σφάλματα και βελτιώνοντας την απόδοση.</p>




<p><strong>Διαχείριση δύο DbContext</strong></p>

<p>Σε εφαρμογές με δύο βάσεις δεδομένων (π.χ. AppDbContext και LoggingDbContext), το Unit of Work μπορεί να συνδυαστεί με TransactionScope για atomicity:<br>
</p>

<div class="highlight js-code-highlight">
<pre class="highlight plaintext"><code>using System.Transactions;

var appContext = new AppDbContext();
var logContext = new LoggingDbContext();

using (var scope = new TransactionScope(TransactionScopeOption.Required,
           new TransactionOptions { IsolationLevel = IsolationLevel.ReadCommitted }))
{
    try
    {
        // Αλλαγές στην κύρια βάση
        var product = new Product { Name = "Laptop", Price = 1200 };
        appContext.Products.Add(product);
        appContext.SaveChanges();

        // Αλλαγές στη βάση logs
        var log = new Log { Message = $"Προστέθηκε προϊόν {product.Name}" };
        logContext.Logs.Add(log);
        logContext.SaveChanges();

        // Commit και στις δύο βάσεις
        scope.Complete();
    }
    catch (Exception ex)
    {
        Console.WriteLine("Σφάλμα: " + ex.Message);
        // rollback γίνεται αυτόματα
    }
}

</code></pre>

</div>



<p><strong>Σημεία κλειδιά:</strong></p>

<ul>
<li>TransactionScope δημιουργεί μια “ambient transaction” για δύο DbContext.</li>
<li>Τα <code>SaveChanges()</code> ενημερώνουν τα context, αλλά commit γίνεται μόνο με <code>scope.Complete()</code>.</li>
<li>Αν υπάρξει σφάλμα, καμία αλλαγή δεν εφαρμόζεται.</li>
</ul>




<p><strong>Best Practices</strong></p>

<ul>
<li>
<strong>Repositories χωρίς SaveChanges()</strong> – commit μόνο από Unit of Work.</li>
<li>
<strong>TransactionScope για πολλαπλά DbContext</strong> για atomicity.</li>
<li>
<strong>Dependency Injection</strong> για DbContext για testability και ευελιξία.</li>
<li>
<strong>Διαχείριση exceptions</strong> γύρω από Complete() για rollback και logging.</li>
<li>
<strong>Σαφής διαχωρισμός:</strong> Repositories για CRUD, Unit of Work για συναλλαγές.</li>
</ul>




<p><strong>Συμπέρασμα</strong></p>

<p>Το Unit of Work είναι εργαλείο που φέρνει <strong>τάξη, συνέπεια και καθαρότητα</strong> στη διαχείριση δεδομένων. Από τα πρώτα χρόνια με αποσπασματικές αλλαγές στη βάση, φτάσαμε σε εφαρμογές όπου όλα τα modifications μπορούν να εκτελούνται ως <strong>ένα ενιαίο έργο</strong>, ακόμα και με πολλαπλά DbContext.</p>

<p>Συνδυάζοντας <strong>Repository Pattern + Unit of Work + TransactionScope</strong>, δημιουργούμε εφαρμογές <strong>καθαρά δομημένες, ασφαλείς και εύκολα συντηρήσιμες</strong>, ιδανικές για σύγχρονες enterprise εφαρμογές.</p>




<p><strong>Βιβλιογραφία</strong></p>

<ol>
<li>Fowler, Martin. Patterns of Enterprise Application Architecture. Addison-Wesley, 2003.</li>
<li>Microsoft Docs. Working with transactions in Entity Framework. <a href="https://learn.microsoft.com/en-us/ef/core/saving/transactions" rel="noopener noreferrer">Link</a>
</li>
<li>Microsoft Docs. Unit of Work and Repository Patterns in .NET. <a href="https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design" rel="noopener noreferrer">Link</a>
</li>
<li>Jimmy Bogard. Effective Aggregate Design. 2009.</li>
<li>Vermeulen, Steven. Entity Framework Core in Action. Manning, 2018. </li>
<li>Lerman, Julie. Programming Entity Framework: Code First. O’Reilly, 2011.</li>
<li>Microsoft Docs. TransactionScope class (System.Transactions). <a href="https://learn.microsoft.com/en-us/dotnet/api/system.transactions.transactionscope" rel="noopener noreferrer">Link</a>
</li>
</ol>

