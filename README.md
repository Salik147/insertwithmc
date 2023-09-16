# insertwithmc
first make database including columns id(increament 1) name designation, image_path
add ado.net entity frame work name model like image.cs
in context.cs files this will appear
namespace WebApplication3.Models
{
    using System;
    using System.Data.Entity;
    using System.Data.Entity.Infrastructure;
    
    public partial class CrudWithImageEntities : DbContext
    {
        public CrudWithImageEntities()
            : base("name=CrudWithImageEntities")
        {
        }
    
        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            throw new UnintentionalCodeFirstException();
        }
    
        public virtual DbSet<Image> Images { get; set; }
    }
}

make a controller named it like homeController
do it as 
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Web;
using System.Web.Mvc;
using WebApplication3.Models;

namespace WebApplication3.Controllers
{
    
    public class HomeController : Controller
    {CrudWithImageEntities db = new CrudWithImageEntities();
        public ActionResult Index()
        {
            var data = db.Images.ToList();
            return View(data);
            
        }

        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Index(int id)
        {
            // Handle the POST request for deletion here
            // You can use the same logic as in the Delete action

            // Find the image record by id
            var image = db.Images.Find(id);

            if (image == null)
            {
                return HttpNotFound(); // Handle the case where the image is not found
            }

            // Remove the image record from the database
            db.Images.Remove(image);
            db.SaveChanges();

            TempData["DeleteMessage"] = "<script>alert('Image has been deleted')</script>";

            // Redirect back to the Index action to display the updated list
            return RedirectToAction("Index");
        }

        public ActionResult About()
        {
            ViewBag.Message = "Your application description page.";

            return View();
        }

        public ActionResult Contact()
        {
            ViewBag.Message = "Your contact page.";

            return View();
        }

        public ActionResult Create() 
        {
            return View();
        }

        [HttpPost]
        public ActionResult Create(Image i) 
        {
            if (ModelState.IsValid == true) 
            {
                string filename = Path.GetFileNameWithoutExtension(i.imagefile.FileName);
                string extension = Path.GetExtension(i.imagefile.FileName);
                HttpPostedFileBase postedFile = i.imagefile;
                int length = postedFile.ContentLength;

                if (extension.ToLower() == ".jpg" || extension.ToLower() == ".jpeg" || extension.ToLower() == ".png")
                {
                    if (length <= 1000000)
                    {
                        filename = filename + extension;
                        i.image_path = "~/Images/" + filename;
                        filename = Path.Combine(Server.MapPath("~/Images/"), filename);
                        i.imagefile.SaveAs(filename);
                        db.Images.Add(i);
                        int x = db.SaveChanges();
                        if (x > 0)
                        {
                            TempData["CreateMessage"] = "<script>alert('Data has been inserted')</script>";
                            return RedirectToAction("Index","Home");
                        }
                        else
                        {
                            TempData["CreateMessage"] = "<script>alert('Data has NOT inserted')</script>";
                        }
                    }
                    else
                    {
                        TempData["sizeMessage"] = "<script>alert('image message should be less than 1mb')</script>";
                    }
                }
                else 
                {
                    TempData["formateMessage"] = "<script>alert('wrong format(not supported)')</script>";
                }
            }
            return View();
        }
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Delete(int id)
        {
            // Find the image record by id
            var image = db.Images.Find(id);

            if (image == null)
            {
                return HttpNotFound(); // Handle the case where the image is not found
            }

            // Remove the image record from the database
            db.Images.Remove(image);
            db.SaveChanges();

            TempData["DeleteMessage"] = "<script>alert('Image has been deleted')</script>";
            return RedirectToAction("Index");
        }
    }
}

in image.cs file add
//------------------------------------------------------------------------------
// <auto-generated>
//     This code was generated from a template.
//
//     Manual changes to this file may cause unexpected behavior in your application.
//     Manual changes to this file will be overwritten if the code is regenerated.
// </auto-generated>
//------------------------------------------------------------------------------

namespace WebApplication3.Models
{
    using System;
    using System.Collections.Generic;
    using System.Web;

    public partial class Image
    {
        public int id { get; set; }
        public string name { get; set; }
        public string designation { get; set; }
        public string image_path { get; set; }

        public HttpPostedFileBase imagefile { get; set; }    
    }
}

in index add this 
<td>
    <img src="@Url.Content(item.image_path)" width="100" height="100" />
</td>

in create.cshtml
<h2>Create</h2>
@Html.Raw(TempData["CreateMessage"])
@Html.Raw(TempData["sizeMessage"])
@Html.Raw(TempData["formatMessage"])

@using (Html.BeginForm("Create", "Home", FormMethod.Post, new { enctype = "multipart/form-data" }))

in create.cshtml add this
<div class="form-group">
    @Html.LabelFor(model => model.image_path, htmlAttributes: new { @class = "control-label col-md-2" })
    <div class="col-md-10">
        <input type="file" required name="imagefile" />
    </div>
</div>

it will look like this 
@model WebApplication3.Models.Image

@{
    ViewBag.Title = "Create";
}

<h2>Create</h2>
@Html.Raw(TempData["CreateMessage"])
@Html.Raw(TempData["sizeMessage"])
@Html.Raw(TempData["formatMessage"])

@using (Html.BeginForm("Create", "Home", FormMethod.Post, new { enctype = "multipart/form-data" }))
{
    @Html.AntiForgeryToken()

    <div class="form-horizontal">
        <h4>Image</h4>
        <hr />
        @Html.ValidationSummary(true, "", new { @class = "text-danger" })
        <div class="form-group">
            @Html.LabelFor(model => model.name, htmlAttributes: new { @class = "control-label col-md-2" })
            <div class="col-md-10">
                @Html.EditorFor(model => model.name, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model => model.name, "", new { @class = "text-danger" })
            </div>
        </div>

        <div class="form-group">
            @Html.LabelFor(model => model.designation, htmlAttributes: new { @class = "control-label col-md-2" })
            <div class="col-md-10">
                @Html.EditorFor(model => model.designation, new { htmlAttributes = new { @class = "form-control" } })
                @Html.ValidationMessageFor(model => model.designation, "", new { @class = "text-danger" })
            </div>
        </div>

        <div class="form-group">
            @Html.LabelFor(model => model.image_path, htmlAttributes: new { @class = "control-label col-md-2" })
            <div class="col-md-10">
                <input type="file" required name="imagefile" />
            </div>
        </div>

        <div class="form-group">
            <div class="col-md-offset-2 col-md-10">
                <input type="submit" value="Create" class="btn btn-default" />
            </div>
        </div>
    </div>
}

<div>
    @Html.ActionLink("Back to List", "Index")
</div>

@section Scripts {
    @Scripts.Render("~/bundles/jqueryval")
}

